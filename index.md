---
layout: default
title: Martin Stevens
---

## About

I'm a security and DevSecOps engineer based in London. I work at a deep technical level across infrastructure, platform, application and security teams, and I like making the secure way of doing something the easiest way.

I also run my own limited company, Budgester Technologies Ltd, which has given me a commercial view of cost, clients and trade-offs.

## What I do

- **Secrets and identity:** secrets management with HashiCorp Vault and CyberArk, and workload identity with SPIFFE/SPIRE in place of static, network-bound credentials.
- **Encryption in transit:** mutual TLS across Kafka and observability platforms.
- **Infrastructure as code:** Ansible, Terraform and OpenTofu.
- **Secure delivery:** security gates built into CI/CD (Bamboo, Bitbucket, GitHub Actions, Argo CD) on OpenShift and Kubernetes.
- **Measuring maturity:** security maturity models mapped to NIST 800-53, so teams can see where they stand and what to do next.

## Background

Sysadmin, then developer, then DevOps, and now security engineering. Most recently I've been the DevSecOps lead on a large infrastructure modernisation programme in financial services.

## Outside work

- **Sailing:** we've sailed the River Crouch in Essex for about 17 years, through three boats: a Westerly Griffon, a Gib'Sea 33 and now *Ragamuffin*, a 2001 Gemini 105MC catamaran kept at Burnham-on-Crouch. We're members of the Royal Burnham Yacht Club. The boat has a Victron energy system reporting over 4G, so of course I monitor it remotely.
- **Hi-fi and home cinema:** a B&W 5.0 speaker set (706 S3 fronts, HTM72 S3 centre, 686 S2 rears) driven by an Arcam AVR5. For vinyl, a J.A. Michell / Transcriptors Hydraulic Reference turntable with a Micro Seiki MA-707 arm, through a Pro-Ject Phono Box S2.

## Posts

{% for post in site.posts %}- {{ post.date | date: "%-d %b %Y" }}: [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}
## Contact

- Email: [budgester@budgester.com](mailto:budgester@budgester.com)
- GitHub: [github.com/budgester](https://github.com/budgester)
- LinkedIn: [linkedin.com/in/budgester](https://www.linkedin.com/in/budgester/)
