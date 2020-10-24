---
layout: post
title: "Cert Review: HashiCorp Certified Terraform Associate"
date: 2020-10-24 14:00:00 +0800
tags: [cert-review]
---

<center>
<img src="/assets/2020-10-24-cert-review-terraform-assoc/sandcastle-571044_640.jpg" />
<br />
<br />
<i>"building sandcastles in the cloud..."</i> (<a href="https://pixabay.com/photos/sandcastle-beach-sand-sea-571044/">source</a>)
</center>
<br />

# Why I did the certification

To learn and solidify my skills in **Infrastructure as Code** (IaC)

# Why you should do it too

- [Terraform](https://www.terraform.io/) is an easy-to-use tool to **provision** and **manage** cloud infrastructure
- **Every** developer managing cloud infrastructure should practice [**Infrastructure as Code**](https://learn.hashicorp.com/tutorials/terraform/infrastructure-as-code) principles.

_Note: Other IaC tools (eg. [CloudFormation](https://aws.amazon.com/cloudformation/), [Google Cloud Deployment Manager](https://cloud.google.com/deployment-manager/docs), [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview)) would be equally effective too 👍_

# Best way to prepare for it

**1. Use the Terraform Exam Review document**

- HashiCorp's very own Terraform [Exam Review document](https://learn.hashicorp.com/tutorials/terraform/associate-review) is very **well-structured** and **comprehensive** 🚀
- I **highly recommend** using the Exam Review document, as it outlines all the important aspects required to pass the exam
- Given that Terraform's scope is not too broad (_as compared to say, AWS_), I think is **not necessary** to purchase a separate structured course.

**2. Do (many) prep tests**

- Same old, same old - **Drill 'em tests!** 🚧

Prep tests:

- [HashiCorp Certified: Terraform Associate Practice Exam 2020](https://www.udemy.com/course/terraform-associate-practice-exam/) by Bryan Krausen
- [HashiCorp Certified: Terraform Associate 2020 Practice Exam](https://www.udemy.com/course/hashicorp-certified-terraform-associate-2020-practice-exam/) by CloudME
- [HashiCorp Certified: Terraform Associate Practice Exam 2020](https://www.udemy.com/course/hashicorp-certified-terraform-associate-practice-exam-2020-mv/) by eBuddy Learning

**3. Practice using it**

- The best way to learn Terraform is to **provision cloud infrastructure** using the Terraform CLI
- Don't forget to explore [Terraform Cloud](https://www.terraform.io/docs/cloud/overview.html)'s features too! (_though it's not covered much in the exam_)
- Since most cloud providers provide a **Free Tier** (eg. [AWS Free Tier](https://aws.amazon.com/free/)), you can practice using Terraform at no cost! 🥳

# Exam Tips & Gotchas

- Things you'll have to memorize:
  - Environment variables used to enable Terraform debugging: `TF_LOG=TRACE` and `TF_LOG_PATH`
  - Variable definition precedence when executing Terraform: [docs](https://www.terraform.io/docs/configuration/variables.html#variable-definition-precedence)
  - Terraform state filenames (when using Terraform Local): `terraform.tfstate.d` and `terraform.tfstate.backup` (for backup state)
  - Terraform tiers feature differences: [docs](https://www.hashicorp.com/products/terraform/pricing)
- Terraform Cloud Workspace vs Terraform Local Workspace
  - Terraform Cloud Workspace: For `remote` backend; stores state of **single** workspace in **single** remote workspace.
  - Terraform Local Workspace: For `local` backend; stores state of **multiple** workspaces in **single** `terraform.tfstate.d` file.
- `terraform show` vs `terraform state`
  - `terraform show`: Displays details of current managed infrastructure.
  - `terraform state`: Used to make manual modifications to state file. Can also display details of current managed infrastructure via `terraform state list`.
- Defining `provider` versions
  - Since Terraform v0.12, it's recommended to define a `provider` version within `required_providers` in `terraform` block. (instead of within a `provider` block)
- `terraform fmt` doesn't format `.tf` files recursively by default

# My Stats

Some stats on myself to give a bit of context on where I was before I took the exam:

**Basic Stats**

| Area                    | Notes                                                                                        |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| Educational Background  | BEng Computer Science degree                                                                 |
| Professional Background | ~3yrs SWE experience, limited cloud exposure, some node.js + docker + k8s. Acquired AWS CSAA |
| Total Prep Time         | 13th August 2020 - 17th September 2020 (~1mth)                                               |
| Exam Taken              | 24th October 2020                                                                            |
| Exam Score              | 91%                                                                                          |

**Practice Test Stats**

<details markdown="1">
<summary>See More</summary>

| Test Try/Score % | 1st | 2nd |
| ---------------- | --- | --- |
| CloudME 1        | 67% | 93% |
| CloudME 2        | 62% | 80% |
| CloudME 3        | 73% | 86% |
| eBuddy 1         | 78% | 92% |
| eBuddy 2         | 89% | 91% |
| eBuddy 3         | 84% | 85% |
| eBuddy 4         | 87% | 96% |
| Actual           | 91% | -   |

</details>
<br />
