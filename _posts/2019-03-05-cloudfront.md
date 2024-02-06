---
title: CloudFront
date: 2019-03-05
layout: post
author: andrew
image: assets/images/cloudfront.svg
categories: [ aws, notes ]
---

CloundFront is Amazon's CDN. It's a system of distributed servers that deliver content to a user based on the geographic location of the user, content origin, and the delivery server.

## Terminology

* Edge Location => Location where content is cached.
* Origin => Origin of all files the CDN will distribute. Can be S3 bucket, EC2 Instance, ELB, or Route53, but does not have to be an AWS service.
* Distribution => Name given to the collection of edge locations
* Web Disbribution => Website distribution
* RTMP Distribution => Media (Adobe Flash) distribution

## How it works

User-A in California wants a video file that's stored in the UK. User-A requests the video from the edge location closest to them. That edge location retrieves and caches the video file from the UK. User-B, also in California, requests the same video file. Instead of retrieving the file from the UK, User-B is able to get the file from the edge location.

## Gotchas

* Edge locations are not just READ only.
* Objects are cached for the life of the TTL
* You can clear cached objects early for $$
* You can have multiple origins in the same distribution
* TTLs are in seconds

