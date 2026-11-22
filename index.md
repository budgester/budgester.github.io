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

## Open source

I've been contributing to open source since 2001. Usually I hit a problem in real work, report it, and send the fix or the missing docs.

- **[IRM (Information Resource Manager)](https://sourceforge.net/projects/irm/):** a developer on this GPL PHP/MySQL asset tracking and helpdesk ticketing system for IT departments, on SourceForge from 2001.
- **[Checkov](https://github.com/bridgecrewio/checkov/pull/4934):** corrected the Bitbucket integration docs to match the code.
- **[The Littlest JupyterHub](https://github.com/jupyterhub/the-littlest-jupyterhub/pull/472):** wrote the AWS Cognito authentication docs.
- **[sonar-cloudformation-plugin](https://github.com/Hack23/sonar-cloudformation-plugin/pull/639):** a Docker install example and clearer setup docs for importing Checkov and cfn-nag results into SonarQube.
- **[Eclipse Che JetBrains dev server](https://github.com/che-incubator/jetbrains-ide-dev-server/pull/66):** error reporting for failed IDE downloads.
- **Bug reports** to [HashiCorp Vault Secrets Operator](https://github.com/hashicorp/vault-secrets-operator/issues/1092), [Sigstore cosign](https://github.com/sigstore/cosign/issues/2611), [StackRox](https://github.com/stackrox/stackrox/issues/17783), Eclipse Che, Ansible and Bandit, among others.
- Earlier: fixes to Chef's knife-rackspace for Windows (WinRM) bootstrapping.

## Outside work

- **Sailing:** we've sailed the River Crouch in Essex for about 17 years, through three boats: a Westerly Griffon, a Gib'Sea 33 and now *Ragamuffin*, a 2001 Gemini 105MC catamaran kept at Burnham-on-Crouch. We're members of the Royal Burnham Yacht Club. The boat has a Victron energy system reporting over 4G, so of course I monitor it remotely. Follow along at [A Shipful of Kittens](https://www.facebook.com/ShipFullOfKittens).
- **Hi-fi and home cinema:** a B&W 5.0 speaker set (706 S3 fronts, HTM72 S3 centre, 686 S2 rears) driven by an Arcam AVR5. For vinyl, a J.A. Michell / Transcriptors Hydraulic Reference turntable with a Micro Seiki MA-707 arm, through a Pro-Ject Phono Box S2.

## Posts

{% for post in site.posts %}- {{ post.date | date: "%-d %b %Y" }}: [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}
## Contact

- Email: [budgester@budgester.com](mailto:budgester@budgester.com)
- GitHub: [github.com/budgester](https://github.com/budgester)
- LinkedIn: [linkedin.com/in/budgester](https://www.linkedin.com/in/budgester/)
