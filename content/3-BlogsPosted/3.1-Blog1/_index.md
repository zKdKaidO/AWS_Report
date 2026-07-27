---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# WHY DOES AWS STILL CHARGE AFTER STOPPING EC2?

A common misunderstanding when learning AWS is: *EC2 is stopped = AWS stops charging.*

However, through posts and messages in the group, I have seen many people still receive bills even when their instance is in the *Stopped* state, or even after it has been *Terminated*.

One real case was a friend of mine who accidentally left a NAT Gateway running after doing a lab and received a bill of nearly $250.

The issue is that EC2 is only one part of the system. The resources created together with it do not always disappear automatically.

## Stopping EC2 only stops virtual machine compute charges

When an EC2 instance is in the Stopped state, AWS stops charging for the compute part. However, the EBS disk is still kept so the machine can be started again later, so the storage capacity is still charged.

A simple way to understand it:

- Stop = Turn off the machine but keep the disk.
- Terminate = Delete the machine, but it is still necessary to check whether the EBS volume is configured to be deleted with the instance.

## NAT Gateway still charges even when EC2 is stopped

NAT Gateway is a separate resource in a VPC, so it does not stop automatically with EC2.

AWS charges based on both:

- The number of hours the NAT Gateway exists.
- The amount of data passing through it.

For example, in AWS pricing for some regions, a NAT Gateway may cost around $0.045 per hour (reference cost), equivalent to more than $32 per month, even with almost no traffic.

If data passes through it, the cost increases further.

## Elastic IP is not free either

If an Elastic IP has been allocated but not released, that address may continue to generate cost.

AWS currently charges for public IPv4 addresses whether they are in use or idle. One address may only cost a few dollars per month, but if many addresses are forgotten across multiple regions, the cost can add up significantly.

## AWS Budget is not a hard spending limit

By default, AWS Budget is mainly used to send alerts. It does not automatically lock the entire account when spending reaches the configured amount.

If the system needs to respond automatically, Budget Actions must be configured, for example:

- Stop a specific EC2 or RDS resource.
- Attach an IAM policy to prevent creating more resources.
- Send alerts through email or SNS.

## Reference checklist after each AWS lab

- Open Cost Explorer and group by Service and Usage Type.
- Check EC2 → Volumes and delete EBS volumes that are no longer needed.
- Check VPC → NAT Gateways.
- Release Elastic IPs that are no longer used.
- Check the correct region because resources in other regions can still generate cost.
- Enable AWS Budget and Cost Anomaly Detection from the beginning.

The most important lesson I learned is: when deleting a service on AWS, do not only ask: *“Have I turned it off?”* Also ask:

*“What supporting resources did this service create, and are they still generating cost?”*

`#AWS` `#EC2` `#CloudComputing` `#AWSCostOptimization`
