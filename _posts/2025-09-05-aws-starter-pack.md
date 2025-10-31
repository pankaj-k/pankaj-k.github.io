---
layout: post
title: AWS based Logstash cluster provisioned with Terraform + GitHub CI/CD
subtitle: Create AWS based Logstash cluster using Terraform with GitHub for CI/CD
cover-img: /assets/img/watercolor_night_logstash_dockers.png
thumbnail-img: /assets/img/CatOnAWS.png
share-img: /assets/img/watercolor_night_logstash_dockers.png
tags: [AWS, Github]
author: Pankaj K
---

This project showcases the deployment of **Logstash** on a **highly available, autoscaling EC2 cluster** behind an **Application Load Balancer**, provisioned end-to-end using **Terraform** and automated with **GitHub Actions** for CI/CD.<!--more-->

While Logstash is used here as the workload example, this infrastructure pattern is **generic and reusable** — you can replace Logstash with any containerized or systemd-based application that needs to scale reliably on AWS.

The solution includes:

- A custom VPC
- Public subnets for the **internet-facing** load balancer
- Private subnets for EC2 instances running Logstash
- An **Autoscaling Group** to automatically manage Logstash capacity
- An ACM-managed TLS certificate to expose a **secure HTTPS endpoint**
- Optional GitHub Actions workflow for automated deployments

Traffic hits the ALB on HTTPS and is forwarded internally to Logstash over HTTP on the private EC2 instances.

More details are in the project README.  
Feel free to fork the repo and adapt it to your environment:

👉 **GitHub Repo:** https://github.com/pankaj-k/basic_vpc_subnet_ec2_loadbalancer


## Terraform Infrastructure

The infrastructure is defined using **Terraform**. Instead of hand-crafting every resource, this project makes use of **community-supported Terraform AWS Modules**, which provide secure and well-maintained best-practice defaults.

Modules used include:

- `terraform-aws-modules/vpc/aws` (VPC, subnets, routing)
- `terraform-aws-modules/autoscaling/aws` (EC2 Autoscaling Group)
- `terraform-aws-modules/alb/aws` (Application Load Balancer)
- `terraform-aws-modules/security-group/aws` (Security Groups)

This keeps the configuration **compact, readable, and reproducible**, while still allowing customization where needed.


## HTTPS + Domain Setup

The domain was purchased via Route 53.  
AWS Certificate Manager (ACM) handles SSL/TLS, and DNS validation is automated using Terraform.

Once applied, the load balancer is available under a **trusted HTTPS endpoint**, ready to receive Logstash input securely.


## GitHub Actions (Optional)

The repository includes an example **GitHub Actions pipeline**.

If you want to automate plan/apply steps:

1. Store AWS `Access Key` + `Secret Key` in GitHub Secrets.
2. Enable the workflow in `.github/workflows/terraform.yml`.

This allows infrastructure changes to be tested, reviewed, and deployed through GitHub.