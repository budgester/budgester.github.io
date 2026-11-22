---
layout: default
title: How this site is written
permalink: /how-this-site-is-written/
---

# How this site is written

This site is a work in progress and keeps changing. The posts come from real work: things I've built, fixed or tidied up, worked through with an AI assistant (Claude Code, running in my terminal) and then written up. This page explains how that works, using the first batch of posts as the example.

## It starts with real work, not a writing prompt

The four posts from November 2026 ([AWS under OpenTofu](/2026/11/bringing-an-aws-account-under-opentofu.html), [the OIDC pipeline](/2026/11/plan-on-pull-request-apply-on-merge-with-github-oidc.html), [the hijacked wildcard](/2026/11/the-wildcard-record-that-gave-away-every-subdomain.html) and [personal domains](/2026/11/managing-a-handful-of-personal-domains.html)) weren't planned as posts. They came out of a couple of days spent sorting out my home setup: NAS updates, Plex, tax paperwork, and then my AWS account and domains.

A typical exchange went like this:

1. **I asked a question or set a goal**, often loosely: "can we manage AWS with Terraform or Ansible?", "bring SES and IAM", "check what davro.net actually points to before fixing".
2. **The assistant investigated first.** It ran read-only commands against the real systems (the AWS CLI, DNS lookups, the GitHub API, `curl`), read my notes and checked email for context, and reported what it found before proposing anything.
3. **It made a recommendation with the reasoning and the trade-offs.** For example: OpenTofu over Ansible for AWS because of plan/diff, imports and drift detection; not using the Google Workspace provider because it's archived; removing old access keys rather than rotating them because nothing needed a long-lived key.
4. **I decided.** Sometimes I agreed, sometimes I went a different way ("leave the email, fix the hijack"), and sometimes I asked for something it hadn't suggested (auto-deleting merged branches, a CI pipeline).
5. **Changes went through the same review as any change**: a branch, a pull request, a plan posted on the PR for me to read, then a merge that applied it. The assistant prepared the PRs; I merged them.

## The guardrails

Working this way only makes sense with limits on what the assistant does on its own:

- **Read freely, change carefully.** Looking things up needed no permission. Anything that changed something, such as deleting IAM policies, deactivating keys, starting DKIM signing or removing a domain from Google Workspace, it proposed first and only did after I said yes.
- **No secrets pass through it.** It doesn't type passwords or handle access keys. When a password needed resetting, it explained the steps and I did it. When new credentials might be needed, I'd create them myself.
- **Plans before applies.** Every infrastructure change was reviewed as an OpenTofu plan in a PR comment before it touched anything.
- **Check before fixing.** Before removing the davro.net records, I asked it to check what they actually served. That's how we found a stranger's website on every subdomain, which turned a tidy-up into a security fix and then into a post.

## It gets things wrong, and that's part of it

The posts include the mistakes, because they're where the useful lessons are:

- It told me DSM 7.3 was the last version my NAS could run. Later that day the NAS was on 7.4.1. It said so plainly and corrected its notes.
- The first CI run failed because GitHub had issued an immutable OIDC subject that the trust policy didn't expect. That failure became half of the [OIDC post](/2026/11/plan-on-pull-request-apply-on-merge-with-github-oidc.html).
- Before deleting two imported IAM policies, it read the provider's source code, found that it doesn't detach a policy before deleting it, and split the change into two pull requests rather than risk a half-finished apply.
- A Todoist call failed because a feature needed a paid plan; it retried without that option and told me.

## From work to post

Writing the posts was the last step, and followed the same pattern:

1. I picked the topics from a list of ideas it had pulled out of the work.
2. It drafted each post from what we had actually done, the real commands, errors and fixes, written in my voice.
3. **It removed anything identifying**: my AWS account ID, the state bucket name, the old server's IP address and my brother's domain name.
4. The drafts went into a pull request as unpublished drafts. I read them, then asked for them to be published, which was another PR.

So the AI does a lot of the legwork: investigating, drafting code, drafting text. The decisions, the review and what gets published are mine. Expect pages to be added, rewritten and occasionally replaced as I learn more.
