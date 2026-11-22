---
title: "Managing a handful of personal domains properly"
---

Over 25 years I've collected eight domains: one for my main email address, a couple used for Google Workspace, a few holding old or never-started projects, and one I look after for a family member. Like a lot of people, I'd registered them, pointed them somewhere and forgotten about them. This post is the checklist I worked through to tidy them up, and what I found.

## 1. Know what you have

Start with an inventory: for each domain, the registrar, the expiry date, whether auto-renew is on, what it's used for and whether it sends or receives mail. I found two domains with auto-renew off that I wanted to keep, one domain still listed in Google Workspace that had lapsed years ago and been re-registered by someone else, and an empty DNS zone left over for a domain I no longer owned.

The lapsed domain is worth dwelling on. Whoever owns it now could receive any mail still addressed to it. Removing it from the Workspace account stopped anything trusting it.

## 2. Put DNS in code

All my zones are now in OpenTofu, and every change goes through a pull request with the plan posted for review (see the earlier posts). The main benefit for personal domains isn't automation: it's that every record is written down, and every change has a reason attached.

## 3. Lock down domains that never send mail

A domain with no email can still be used to send spoofed email, unless it says so. Two TXT records do it:

```
example.org.         TXT  "v=spf1 -all"
_dmarc.example.org.  TXT  "v=DMARC1; p=reject; sp=reject; adkim=s; aspf=s"
```

SPF `-all` says no server may send as this domain, and DMARC `p=reject` tells receivers to reject anything that claims to come from it. Four of my domains got these.

## 4. Authenticate the domains that do send mail

For the Google Workspace domains:

- **SPF:** `v=spf1 include:_spf.google.com ~all`
- **DKIM:** generate a 2048-bit key in the Workspace admin console (Gmail, then Authenticate email), publish it as a TXT record at `google._domainkey`, and only then click *Start authentication*. Without your own key, Google signs with a default domain that doesn't match yours, so DMARC relies on SPF alone, and forwarded mail fails.
- **DMARC:** start with `p=none` and a report address, read the reports for a few weeks, then move to `p=quarantine` and later `p=reject`.

Two practical details:

- **Long DKIM keys don't fit in one TXT string.** A 2048-bit key is over 255 characters, so it has to be split into two quoted strings in the same record, which DNS joins back together.
- **Reports for other domains need permission.** If `example.net` sends its DMARC reports to an address at `example.com`, then `example.com` needs a record at `example.net._report._dmarc.example.com` containing `v=DMARC1`, or receivers won't send them.

## 5. Make the website match

My main domain's website had been a Blogger blog on `www` since about 2018, HTTP only, and the bare domain didn't load at all. It's now GitHub Pages, with:

- A and AAAA records for the apex, and `www` as a CNAME to the Pages site.
- A **verified domain** on my GitHub account, so no other account can publish a Pages site on it.
- A CAA record limiting certificates to Let's Encrypt.
- The old blog posts at their original URLs, so existing links still work.

## 6. Check for things pointing at nothing

Any record pointing at a server, bucket or service you no longer control is a liability. I found a wildcard record sending every subdomain of one domain to a stranger's server (written up separately). Look for records pointing at raw IPs and CNAMEs to services you've cancelled.

## 7. Write it down

The last step was a single page in my notes: which domains exist, what each is for, how mail flows (which turned out to be different from how I thought it worked), what sends mail, and the rules for adding records. A year from now I'll have forgotten all of this, and that page is what I'll read.

## The checklist

- [ ] Inventory: registrar, expiry, auto-renew, purpose
- [ ] Remove anything that trusts a domain you no longer own
- [ ] DNS in code, changed by reviewed pull request
- [ ] SPF `-all` and DMARC `p=reject` on domains that don't send mail
- [ ] SPF, DKIM and DMARC on domains that do; tighten DMARC over time
- [ ] CAA records
- [ ] Verified domains on hosting platforms
- [ ] No records pointing at things you don't control
- [ ] A page of notes explaining it all
