---
date: 2026-11-22 11:00:00 +0000
title: "The wildcard DNS record that gave away every subdomain"
---

While [importing my DNS into OpenTofu](/2026/11/bringing-an-aws-account-under-opentofu.html), I found two odd records on a family member's domain that I look after:

```
*.example.net   A   <an old cloud server IP>
@.example.net   A   <the same IP>
```

The second one is a record literally named `@`, which was probably meant to be the zone apex. Both pointed at a cloud server we'd stopped using years ago. The main site and `www` were fine: they pointed at GitHub Pages. Before changing anything, I checked what those records actually did on the live internet.

## What it served

```sh
$ curl -s http://anything.example.net/ | grep -o '<title>[^<]*'
<title>Remotebelasting Inkomebronne vir Suid Afrikaners in 2025
```

Any subdomain at all, `anything.`, `test123.`, whatever you typed, showed **a stranger's website**: an Afrikaans page about remote income for South Africans. The IP belonged to the same cloud provider, but the server had long since been handed to someone else, and its reverse DNS pointed at an unrelated domain.

Nobody had attacked anything. We released a server, the provider reused its IP address, and a wildcard record nobody remembered sent every subdomain to the new owner. This is usually called **dangling DNS**, and it's a common route to subdomain takeover.

## Why it matters

- **Phishing that looks like you.** `login.example.net` or `invoice.example.net` would show whatever the new owner chose, under your domain name.
- **Valid HTTPS certificates.** Whoever controls the server can pass the usual domain-validation checks for any name the wildcard covers, and get a trusted certificate for it, unless something stops them.
- **Reputation.** Search engines and mail filters judge your domain partly on what's served under it.

## The fix

Three changes, made in one pull request with the plan reviewed first:

1. **Delete the wildcard and the `@` records.** Now `anything.example.net` returns NXDOMAIN.
2. **Add a CAA record**, so only the certificate authority actually in use can issue certificates, and nobody can issue wildcard certificates:

   ```
   example.net.  CAA  0 issue "letsencrypt.org"
   example.net.  CAA  0 issuewild ";"
   ```

   GitHub Pages uses Let's Encrypt, so the real site keeps working.
3. **Leave everything else alone**, including mail, which needed its owner's input.

Then I checked again: subdomains returned NXDOMAIN from public resolvers, the real site still loaded, and the CAA record was live. And I emailed the domain's owner to explain what I'd changed and why.

## How to check your own domains

- **List every record** and ask what each one is for. Wildcards and records pointing at raw IP addresses deserve the most suspicion.
- **For each IP, ask whether you still control it.** Cloud IPs are recycled. If you've deleted the server, the record should go too.
- **Request a random subdomain** (`curl http://random-name.yourdomain/`) and see what answers.
- **Add CAA records**, even on domains with no website.
- **Do the same for CNAMEs** pointing at services you've cancelled: old storage buckets, Pages sites and app platforms are the classic takeover targets.
- **Verify your domain** with hosting platforms that support it. GitHub Pages lets you verify a domain on your account, so no other account can publish a site on it.

Having DNS in code helped here: the oddity stood out because I was reading every record while generating the code, and the fix was a reviewed, recorded change rather than a quick click in a console.
