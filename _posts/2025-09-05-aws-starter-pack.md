---
layout: post
title: AWS basic infra
subtitle: AWS infrastructure using Terraform with GitHub for CI/CD
cover-img: /assets/img/African_grasslands.jpeg
thumbnail-img: /assets/img/start_here.jpg
share-img: /assets/img/African_grasslands.jpeg
tags: [AWS, Github]
author: Pankaj K
---

# AWS starter pack

This is about as classic as it gets for the AWS beginners. A simple setup with VPC, Subnets, Routing tables, EC2 instances, Load balancer and Autoscaling. More details in the README in source folder. Feel free to the [GitHub Repo](https://github.com/pankaj-k/basic_vpc_subnet_ec2_loadbalancer) as starting point.


## Terraform

The infrastructure code is in Terraform. However hand coding everything quickly leads to a big code base. Leverage not Re-Invent. So this code repo uses the already created [Terraform modules](https://registry.terraform.io/namespaces/terraform-aws-modules) for different AWS functionality. VPC, EC2, Loadbalancer and Autoscaling are implemented via Terraform modules.

## GitHub Actions

If you want to use GitHub Actions to compile and deploy code then that is possible. Make sure you follow the README in the source folder to set up the GitHub with AWS Secrets. 