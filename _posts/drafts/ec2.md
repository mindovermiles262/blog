---
title: Elastic Compute Cloud (EC2) 
date: 2019-03-13
layout: post
author: andrew
image: assets/images/ec2.png
categories: [ aws, notes ]
---

# Elastic Compute Cloud (EC2)

EC2 is a web service that provides resizable compute capacity in the cloud. EC2 reduces the time required to obtain and boot a new compute device. With EC2 you're only charged for what you use.

# Pricing

* On Demand => Allows you to pay a fixed rate by the time period. No commitment
* Reserved Instances => Contract with Amazon for between 1-3 years with significant discount over On-Demand pricing.
* Spot => Set your own start/stop prices.
* Dedicated Hosts => Physical EC2 servers dedicated for your use

### On Demand

Good for people that want low cost and flexibility on AWS without up-front payment or long-term commitment. Also good for applications with short-term, spiky or unpredictable workloads that cannot be interrupted.

### Reserved Instances

Useful for applications with steady state, predictable uses (web servers). Users make up front payments to reduce computing costs even further. Up to 75% off on-demand.

Convertible Reserved Instances => You're able to change reservations so long as of greater or equal value.

Scheduled RI => Allows you to set capacity reservations to match predictable schedule, such as an end of month sale or weekly data processing.

### Spot

You set your start and stop prices. When spot pricing dips below your start price, EC2 instances are provisioned until the spot price increases above your stop price. Good for applications that allow for flexible start and end times with the need for very low compute prices.

You are not charged for the last (partial) hour of the instance if AWS terminates the instance. You will be charged for the entire hour if YOU terminate the instance.

### Dedicated Hosts

Useful for regulatory requirements that do not support multi-tenet virtualisation, or for Licensing constraints. Dedicated Hosts can be purchased on-demand or reserved capacities.

# Instance Types

| Family  | Role | Use Cases |
|---------|------|-----------|
|Fx       | Field Programmable Gate Array | Genomics Research, Financial Analysis, Real-Time Video Processing, Big Data |
|Ix       | High Speed Storage | NoSQL, DBs, Data Warehousing |
|Gx       | Graphics Intensive | Video Encoding, 3D Application Streaming |
|Hx       | High Disk Throughput | MapReduce-based workloads, distributed file systems (HDFS & MapR-FS) |
|Tx       | Low cost, General Purpose | Web Servers, Small DB's |
|Dx       | Density Storage | File servers, Data Warehousing, Hadoop |
|Rx       | Memory Optimized (**R**AM) | Memory intensive applications and DB's |
|Mx       | General Purpose | Application Servers
|Cx       | Compute Optimized | CPU Intensive applications and DB's |
|Px       | Graphics, General Purpose GPU | Machine Learning, Bitcoin mining |
|Xx       | Memory Optimized (**X**treme RAM) | SAP HANA, Apache Spark |

### FIGHT DR MC PX

![Screenshot](https://imgur.com/b25hKDt.png)


# Elastic Block Storage (EBS)

Allows you to create storage volumes and attach them to EC2 instances. EBS Volumes are placed in specific AZs where they are automatically replicated to protect you from failure.

You can copy EBS images to different AZs by stopping in instance then clicking "Create Snapshot" from the EC2 Console window. Then you can deploy that image to a new subnet or to a different region (Use the "Copy AMI" from the Actions Menu) under Images > AMIs.

Snapshots of EBS exist on S3 and are incremental. Only changes to the volume are saved after the initial snapshot.

EBS Volumes will **ALWAYS** be in the same AZ as the EC2 instance. The latency would be too much for any other setup.

Snapshots of encrypted volumes are automatically encrypted. Volumes restored from encrypted snapshots are encrypted. Encrypted snapshots CANNOT be shared.

Your root volume EBS will be deleted by default, but other (additional) volumes will not be deleted automatically. You will be charged for these in the free tier

### Types of EBS Volumes

* General Purpose SSD (GP2)
  * General Purpose - Balances price and performance
  * Good for < 10,000 IOPS

* Provisioned IOPS SSD (IO1)
  * Designed for I/O Intensive applications such as RDBs and NoSQL DBs
  * Use if you need > 10,000 IOPS
  * Provisionable up to 20,000 IOPS per volume

* Throughput Optimized HDD (ST1)
  * Used with Big Data, Data Warehousing, Log Processing
  * Cannot be boot volume

* Cold HDD (SC1)
  * Lowest cost storage.
  * Infrequently accessed data such as file servers.
  * Cannot be boot volume

* Magnetic (Standard)
  * Lowest cost bootable
  * Ideal for infrequently accessed data where low storage cost is important

# Security Groups

SGs are "Virtual Firewalls". Each EC2 instance can have multiple security groups. SGs are **Stateful**

All inbound traffic is blocked by default. Outbound traffic is allowed. Changes take effect immediately. You cannot block specific IP Addresses using Security Groups (Use Network ACLs instead)

You can specify allow rules, but not deny rules.

# IAM Roles

IAM roles can be defined and attached to instances. You can now add/edit/remove IAM Roles to EC2 instances on the fly. Role changes take effect immediately.

The use of IAM Roles are more secure than using credentials because nothing is being saved locally (using credentials, key and secret are stored in ~/.aws/credentials)

# Amazon Machine Images (AMI)

AMIs are used most often to take "snapshots" of existing root volumes for the purpose of re-provisioning with encryption.

### Creating an AMI

First, stop the EC2 instance. Then go into the Volumes tab and choose "Create Snapshot." It will then save it as a snapshot which you can then either copy to a new region, create a new volume (which you can encrypt) from that snapshot, or create an image from the snapshot.

# Using the CLI

The AWS CLI is installed by default on the AWS Linux AMIs. You can also install it locally on your computer via the [documentation](https://aws.amazon.com/cli/).  You will need to have access to a CLI User (set up in IAM).

To use some CLI commands across regions you may need to specify the `--region` flag on the command you're trying to run. This region is the location where the resource is located:

```bash
$ aws s3 cp s3://my-bucket/folder/file.jpg ~/Pictures --region us-west-2
```

Basic Usage: 

* `aws configure` => Set up user id and secret for terminal (CREDS STORED UNENCRYPTED in ~/.aws dir !!)
* `aws <service> help` => Help based on the service (S3, EC2, etc.)
* `aws s3 ls` => List all S3 buckets on your account.
* `aws ec2 describe-instances` => List information about ALL EC2 instances. 

# Ephemeral Storage (Instance Store)

Ephemeral Storage are storage devices that persist only to the EC2 instance. Should an instance go down, all data stored in Ephemeral Storage disks is lost. ES is used for applications which require super-fast I/O. The ES disks are physically attached to the hosts in the AWS data centers and, thus, have minimal latency.

Inside of the EC2 instance, you cannot start or stop the instance if using Ephemeral Storage. Should the AWS hypervisor fail, you will lose your ES

# Elastic Load Balancers (ALB, ELB)

Elastic Load Balancers manage the traffic between many web servers. Inside of AWS there are 3 types of load balancers:

1. Application Load Balancers:
  * Best suited for load balancing HTTP(S) traffic.
  * Operate at Layer 7, Application aware balancing.
  * Intelligent - You can create custom routing rules

2. Network Load Balancers (Not part of CSAA)
  * Best suited for load balancing TCP traffic where extreme performance is required.
  * Operate at Layer 4
  * Able to handle millions of requests per second with ultra-low latencies

3. Classic Load Balancers (Deprecated)
  * Balance HTTP(S) applications
  * Use layer 7-specific features such as `X-Forwarded-For` and sticky sessions.
  * Able to use layer 4 routing for TCP-only applications

Classic ELBs work with health checks. Health checks pass with HTTP 200 responses on a specific file (which you can set up, typically `index.html`). You can also configure the timeout, retries, and rest periods of the health checks.

Newer-generation ALBs use routing targets across Subnets. You set up "Target Groups" to specific which instances and protocols to route to, along with the path, threshold, retry, and intervals for health checks.

EC2 instances are reported as "In Service" or "Out of Service" by ELBs.

You are never given an IP address of a load balancer. Instead, use the provided DNS address

# Cloud Watch

Cloud watch is a feature that monitors your EC2 instances. Using "Standard Monitoring" checks the status of the server every 5 minutes. "Detailed Monitoring" will give you statuses every minute.

You can create custom dashboards inside of Cloud Watch, such as CPU utilization, Disk usage, etc. RAM monitoring requires a custom script (outside the scope of CSAA)

You can also set alarms that notify you when a particular event occurs or thresholds are hit. These alarms are set via "Events" which help you respond to changes in your AWS resources.

Cloudwatch will also log any changes - helping you aggregate, monitor, and store logs. "CloudTrail" is a different AWS resource which is for Auditing of your entire AWS environment (Creating new users, New S3 buckets, etc.).

# Metadata

We use a simple `curl` url to gather information about your EC2 instance:

```
# Listing of available parameters:
curl http://169.254.169.254/latest/meta-data/

# Get EC2 Instance Meta Data:
curl http://169.254.169.254/latest/meta-data/public-ipv4
```

# EFS (Elastic File System)

EFS is a file storage service for EC2 instances. It provides a simple interface to create and configure file systems. EFS, storage is dynamic. It grows and shrinks automatically so your applications have the storage they need, when they need it.

EFS grows on EBS volumes, but allows you to attach 2+ EC2 Instances. It uses BLOCK-BASED Storage.

### Features

* Supports NFSv4 Protocol
* You only pay for what you use
* Can Scale up to petabytes
* Supports thousands of NFS Connections
* Data stored across multiple AZs within a region
* Read after Write Consistancy

# Exam Hints

* Termination Protection is OFF by default.
* On an EBS-backed instance, the default action is for the root volume to be deleted with the instance
* EBS Root Volumes on your DEFAULT AMI's cannot be encrypted.
  * You can use 3rd party tools (BitLocker, etc.) to encrypt drives
  * You can make a copy of an AMI and encrypt it
* Additional volumes can be encrypted
* Read the [ELB FAQ](https://aws.amazon.com/elasticloadbalancing/faqs/) for Classic Load Balancers

