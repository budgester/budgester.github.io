---
title: "Bringing an existing AWS account under OpenTofu"
---

My personal AWS account had been run by hand for over ten years: eight domains in Route 53, an SES identity for email, a couple of IAM users and some policies from 2015 that nobody remembered creating. It worked, but every change was a click in the console with no record of why, and no way to see what a change would do before making it.

So I moved it all into [OpenTofu](https://opentofu.org/), the open-source fork of Terraform. This post covers how I did that without recreating anything, and the gotchas I hit on the way.

## The goal: a first plan that changes nothing

The rule I set myself was that the first `tofu plan` after importing had to say:

```
Plan: 31 to import, 0 to add, 0 to change, 0 to destroy.
```

If the plan wants to change something on day one, then either the code doesn't describe reality yet, or you're about to make a change you didn't intend. Neither is a good start.

## State first

Before any resources, I created an S3 bucket by hand for the state: versioned, encrypted, public access blocked, and a bucket policy that refuses anything not over TLS. OpenTofu's S3 backend now has native locking with a lock file next to the state, so there's no DynamoDB table to manage:

```hcl
backend "s3" {
  bucket       = "my-tofu-state-bucket"
  key          = "aws/terraform.tfstate"
  region       = "eu-west-2"
  encrypt      = true
  use_lockfile = true
}
```

The bucket is the one thing that isn't managed as code. Something has to exist before the code can run, and keeping that to a single bucket keeps the bootstrap small.

## Generate the code from what's really there

Writing 23 DNS records by hand is how typos get into production. Instead, I dumped every hosted zone with the AWS CLI and wrote a short Python script that turned the JSON into two files:

- `route53.tf`: a `locals` map of every record, and one `aws_route53_record` resource with `for_each` over it.
- `imports.tf`: matching `import` blocks, also using `for_each`.

```hcl
import {
  for_each = local.import_record_ids
  to       = aws_route53_record.this[each.key]
  id       = each.value   # "ZONEID_name_TYPE"
}
```

`import` blocks (rather than the old `tofu import` command) mean the import is part of the plan: you see it, review it and apply it like any other change. Once it's applied, the file can be deleted.

## The gotchas

The first plans weren't clean, and each difference taught me something.

**Empty zone comments.** Four of my hosted zones had no comment. If you set `comment = null`, the AWS provider fills in its default, "Managed by Terraform", and wants to update the zone. Setting `comment = ""` keeps it empty.

**Tabs inside MX records.** Two domains had MX values with a tab between the priority and the host (`1<TAB>ASPMX.L.GOOGLE.COM.`), presumably from a copy and paste years ago. Generating the code from the live data kept the tab (`"1\tASPMX..."` in HCL), so the plan matched. Retyping them would have shown a change.

**Escaped names.** Route 53 returns some characters as octal escapes: a wildcard comes back as `\052.example.com`, and an `@` as `\100`. The script had to unescape those, both in the record names and in the import IDs. One of them turned out to be a literal record called `@.example.com`, probably someone's attempt at the zone apex. More on that in another post.

**Default tags.** I'd added `default_tags` to the provider, which made every zone show a tag change. I took them out for the import, so the first apply really changes nothing. Tags can come later as a deliberate change.

## IAM and SES

Once DNS was in, the same approach worked for IAM users, groups and policies, and the SES domain identity. Two rules made this safer:

- **No secrets in the code.** Access keys, console passwords and MFA devices stay out of OpenTofu, because anything it manages ends up in the state file in plain text. Users and policies are code; credentials are created by hand.
- **Clean up in separate steps.** I found five policies attached directly to my admin user that duplicated what the admin group already gave it, plus two 2015 policies and a role for a retired AWS service. Deleting them taught me that the AWS provider doesn't detach a policy before deleting it, and because the resources were imported, OpenTofu had no recorded dependency to do the detach first. So I detached in one change and deleted in the next.

## What I'd do again

- Make the first plan show zero changes, whatever it takes.
- Generate the code from live data; don't retype it.
- Keep credentials out of the state.
- Go one area at a time: DNS first, then IAM and SES, then clean up.

The next step was making sure nobody, including me, changes anything without a reviewed plan. That's the [pipeline post](#).
