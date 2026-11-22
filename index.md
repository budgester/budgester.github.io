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

## Writing

{% for post in site.posts limit:5 %}- [{{ post.title }}]({{ post.url | relative_url }}) <small>({{ post.date | date: "%b %Y" }})</small>
{% endfor %}
[All posts]({{ '/blog/' | relative_url }})

## About this site

This site is a work in progress and is constantly evolving. The content is based mainly on things I work on, and also on technical discussions I have with AI, which I use as a sounding board to explore ideas, check my thinking and try things out before writing them up. Expect pages to be added, rewritten and occasionally replaced as I learn more.

## Contact

- Email: [budgester@budgester.com](mailto:budgester@budgester.com)
- GitHub: [github.com/budgester](https://github.com/budgester)
