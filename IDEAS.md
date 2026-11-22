# Ideas for future posts

Not published (excluded in `_config.yml`). Add ideas here; when one becomes a post, move it to `_posts/` and remove it from this list.

## About the site

- **How this site is written.** It's a work in progress and keeps evolving. The content comes mainly from things I work on, and also from technical discussions with AI: using it as a sounding board to explore ideas, check my thinking and try things out before writing them up.

## From work on the home setup (Nov 2026)

- **Bringing an existing AWS account under OpenTofu:** importing Route 53, IAM and SES with `import` blocks so the first plan shows no changes; generating the code from live data; the gotchas (empty zone comments becoming "Managed by Terraform", tab characters in MX values, octal-escaped names like `\052`).
- **Plan on pull request, apply on merge, with no stored AWS keys:** GitHub Actions OIDC roles, one usable only from pull requests and one only from `main`; GitHub's immutable OIDC subject (`repo:owner@id/repo@id:...`) breaking the usual trust policy.
- **Dangling DNS:** a forgotten wildcard record pointing at an old DigitalOcean IP that now serves a stranger's site on every subdomain; how to find and fix it, and why CAA records matter.
- **Email authentication for a handful of personal domains:** SPF, DKIM and DMARC for Google Workspace domains, and locking down domains that never send mail (`-all`, `p=reject`).
- **Deleting imported IAM policies:** the provider doesn't detach before deleting, and imported resources have no recorded dependencies, so detach and delete in separate changes.
- **Replacing long-lived access keys** with `aws login` and MFA, and finding what still uses an old key.

## Older drafts

- **Brownfield DevOps:** `docs/brownfield-devops.md` (OODA loop, "what is your pain point?"). Needs tidying before publishing.
