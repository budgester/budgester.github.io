---
date: 2026-11-22 00:02:00 +0000
title: "Plan on pull request, apply on merge, and GitHub's immutable OIDC subject"
---

Once my AWS account was [described in OpenTofu](/2026/11/bringing-an-aws-account-under-opentofu.html), I wanted a rule: every change goes through a pull request, the plan is posted on the PR for me to read, and only merging to `main` applies it. No AWS keys stored in GitHub, and no way for a branch to apply anything.

This post covers how that's set up, and the one thing that broke it on the first run: GitHub's newer, immutable OIDC subject claim.

## No stored keys: OIDC

GitHub Actions can ask GitHub for a short-lived, signed OIDC token that says which repository, branch or event a job is running for. AWS can trust GitHub as an identity provider and swap that token for temporary credentials. No long-lived access key exists anywhere.

```
pull request / push to main
  → GitHub Actions job asks GitHub for an OIDC token
  → aws-actions/configure-aws-credentials calls STS AssumeRoleWithWebIdentity
  → AWS checks the role's trust policy against the token
  → temporary credentials (1 hour max)
  → tofu init / plan / apply
```

## How the job asks GitHub for a token

Nothing in the repository holds a credential. The token is minted fresh for each job, and the job has to be allowed to ask for it.

**1. Give the workflow permission.** By default a job can't request an OIDC token. One line in the workflow allows it:

```yaml
permissions:
  contents: read
  id-token: write       # allow this job to request an OIDC token
  pull-requests: write  # to post the plan as a comment
```

**2. The runner provides a request URL.** With that permission, GitHub puts two environment variables into the job: `ACTIONS_ID_TOKEN_REQUEST_URL` and `ACTIONS_ID_TOKEN_REQUEST_TOKEN`. The second is a short-lived bearer token that's only good for asking for the first. `aws-actions/configure-aws-credentials` uses them for you, but this is what it does underneath:

```sh
curl -s -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
  "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=sts.amazonaws.com" | jq -r .value
```

The `audience` parameter becomes the token's `aud` claim. AWS expects `sts.amazonaws.com`, and the trust policy checks for it.

**3. GitHub returns a signed JWT.** Its claims describe exactly where the job is running. Decoded, the interesting parts look like this for a pull request:

```json
{
  "iss": "https://token.actions.githubusercontent.com",
  "aud": "sts.amazonaws.com",
  "sub": "repo:OWNER@12345/REPO@67890:pull_request",
  "repository": "OWNER/REPO",
  "event_name": "pull_request",
  "ref": "refs/pull/4/merge",
  "exp": 1795300000
}
```

On a push to `main`, `sub` ends in `:ref:refs/heads/main` and `event_name` is `push`. The token expires within minutes.

**4. Swap it for AWS credentials.** The action calls STS with the token and the role it wants. The equivalent CLI call is:

```sh
aws sts assume-role-with-web-identity \
  --role-arn arn:aws:iam::ACCOUNT:role/gh-aws-plan \
  --role-session-name gh-run \
  --web-identity-token "$TOKEN"
```

AWS checks the signature against GitHub's published keys (that's what the OIDC provider registered in IAM is for), then checks the role's trust policy against `aud` and `sub`. If they match, it returns temporary credentials, which the action exports as environment variables for the rest of the job.

To see the claims your own jobs get, decode the token in a throwaway step. Don't print the token itself in a public log: it's a credential until it expires.

## Two roles, chosen by event

- **Plan role:** read-only access to the account, plus read access to the state and its lock file. It can only be used from **pull requests**.
- **Apply role:** can change things. It can only be used from the **`main` branch**.

The workflow picks the role from the event:

{% raw %}
```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-region: eu-west-2
    role-to-assume: ${{ github.event_name == 'push'
      && 'arn:aws:iam::ACCOUNT:role/gh-aws-apply'
      || 'arn:aws:iam::ACCOUNT:role/gh-aws-plan' }}
```
{% endraw %}

The important part is that **AWS** enforces which role each event can have, not the workflow file. Even if someone edited the workflow on a branch to ask for the apply role, AWS would refuse, because the token says `pull_request`, not `main`. That matters to me because I'm on GitHub's free plan, which doesn't offer branch protection on private repositories. The PR-only rule is a habit there, but the "branches can't apply" rule is enforced by AWS.

## The trust policy

The trust policy checks two claims in the token: the audience (`aud`) is AWS STS, and the subject (`sub`) is exactly the repository and event allowed:

```hcl
condition {
  test     = "StringEquals"
  variable = "token.actions.githubusercontent.com:sub"
  values   = ["repo:OWNER/REPO:pull_request"]   # plan role
}
```

For the apply role, the subject is `repo:OWNER/REPO:ref:refs/heads/main`.

## The first run failed

The first pull request failed at the sign-in step:

```
Could not assume role with OIDC: Not authorized to perform sts:AssumeRoleWithWebIdentity
```

Every guide I'd seen used the `repo:OWNER/REPO:...` format. But GitHub now offers an **immutable subject** format, and my new repository was using it:

```sh
$ gh api repos/OWNER/REPO/actions/oidc/customization/sub
{"use_default":true,"use_immutable_subject":true,
 "sub_claim_prefix":"repo:OWNER@12345/REPO@67890"}
```

The subject includes the numeric **owner ID and repository ID**. So the token's `sub` was `repo:OWNER@12345/REPO@67890:pull_request`, which didn't match my trust policy.

This is a good change. With names only, if you renamed or deleted a repository and someone else created one with the same name, their workflows would match your trust policy. IDs don't get reused, so the immutable form closes that gap.

## The fix

I made the trust policies accept both forms, with the immutable one first:

```hcl
locals {
  github_subjects = ["repo:OWNER@12345/REPO@67890", "repo:OWNER/REPO"]
}

values = [for p in local.github_subjects : "${p}:pull_request"]
```

The roles had to be updated from my laptop, because CI couldn't sign in to fix itself, but after that the pull request plan went through. In hindsight I'd accept only the immutable form, and check the format with that `gh api` call before writing the trust policy.

## Posting the plan on the PR

The last step posts the plan as a comment on the pull request, and updates the same comment on each push instead of adding new ones:

```yaml
- uses: actions/github-script@v7
  with:
    script: |
      const marker = '<!-- tofu-plan -->';
      // find an existing comment starting with the marker; update it or create one
```

Every change now looks the same: open a branch, push, read the plan in the PR comment, merge, and watch the apply run on `main`.

## Bootstrapping

One chicken-and-egg problem: the OIDC provider and the two roles have to exist before CI can use them. I applied those once from my laptop, and everything since has gone through pull requests.
