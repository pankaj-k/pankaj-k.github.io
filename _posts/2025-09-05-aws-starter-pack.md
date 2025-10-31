---
layout: post
title: Logstash running on Autoscaled EC2 cluster
subtitle: AWS infrastructure using Terraform with GitHub for CI/CD
cover-img: /assets/img/African_grasslands.jpeg
thumbnail-img: /assets/img/start_here.jpg
share-img: /assets/img/African_grasslands.jpeg
tags: [AWS, Github]
author: Pankaj K
---

# AWS starter pack

This is post is all about running Logstash a group of Autoscaled EC2 instances. It is a simple setup.
It will create a with VPC with public subnets for load balancer and private subnet for the EC2 instances.
The EC2 instances will be created as a part of Autoscaling group.
I have bought a domain from AWS itself so that I can have a https endpoint for loadbalancer.
The traffic sent to this endpoint will be then moved to the EC2 instances for logstash.

More details in the README in source folder. Feel free to fork the [GitHub Repo](https://github.com/pankaj-k/basic_vpc_subnet_ec2_loadbalancer) and use it as starting point for your scalable Logstash cluster.



## Terraform

The infrastructure code is in Terraform. However hand coding everything quickly leads to a big code base. Leverage not Re-Invent. So this code repo uses the already created [Terraform modules](https://registry.terraform.io/namespaces/terraform-aws-modules) for different AWS functionality. VPC, EC2, Loadbalancer and Autoscaling are implemented via Terraform modules.

## GitHub Actions

If you want to use GitHub Actions to compile and deploy code then that is possible. Make sure you follow the README in the source folder to set up the GitHub with AWS Secrets. 