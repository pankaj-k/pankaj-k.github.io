---
layout: post
title: Logstash on Dockers
subtitle: Part 1 : Create Docker image and upload it to AWS ECR
cover-img: /assets/img/watercolor_night_logstash_dockers.png
thumbnail-img: /assets/img/aws_fargate.png
share-img: /assets/img/path.jpg
tags: [Logstash, AWS, ECR, ECS, Docker, Github]
author: Pankaj K
---

We will do this whole mini-project in small steps. Otherwise it will seem like a big lump of code which magically works. 
The end product will be a cluster of Logstash servers behind a loadbalancer listening to the incoming http traffic. 
To keep this one simple I am making using http input for logstash but you can use any input. You might not need the loadbalancer in many of the cases, like using JDBC input. 

We will also configure the ECS Fargate to resize the cluster depending on the load. And to finish it off we will have some aws based observability around the whole thing. 

Sounds boring? You bet. Lets dig in.