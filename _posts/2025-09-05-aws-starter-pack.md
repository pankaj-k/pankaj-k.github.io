---
layout: post
title: AWS Infrastructure Provisioned with Terraform + GitHub CI/CD
subtitle: AWS infrastructure using Terraform with GitHub for CI/CD
cover-img: /assets/img/African_grasslands.jpeg
thumbnail-img: /assets/img/start_here.jpg
share-img: /assets/img/African_grasslands.jpeg
tags: [AWS, Github]
author: Pankaj K
---

## Overview

This project demonstrates how to deploy **Logstash** on a **highly-available, autoscaled EC2 cluster** behind an **Application Load Balancer**, using **Terraform** for infrastructure provisioning and **GitHub Actions** for CI/CD.

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