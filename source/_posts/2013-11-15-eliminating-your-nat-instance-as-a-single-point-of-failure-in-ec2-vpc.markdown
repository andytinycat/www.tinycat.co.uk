---
layout: post
title: "Eliminating your NAT instance as a single point of failure in EC2 VPC"
date: 2013-11-15 19:51
comments: true
categories: sysadmin ec2
---

Amazon's VPC is wonderful, allowing you to push a section of your internal network into the cloud, with full control over
routing, addressing, subnetting, etc. 

There's one obvious sore point, which is that by default, VPC machines do not get a public IP address like their EC2 Classic
counterparts. The upshot of this is that they cannot access the Internet without what Amazon refer to as a NAT instance.

## NAT instances

All a NAT instance is is a single EC2 instance with IP forwarding enabled, the source/dest check disabled, and `iptables` 
providing NAT using the masquerade table. However, you can only have a single NAT instance - making it a single point of
failure. You can create multiple instances, and switch between them by editing the VPC routing table, but that requires
manual intervention.

At Forward3D, we have a number of instances that need to be able to talk to the Internet, but don't need to be talked to
from the Internet, making NAT a perfect fit. I wanted to eliminate the NAT instance as a SPOF; if these had been physical
machines, I would have used Pacemaker & Corosync to share a VIP (Virtual IP address) between two machines - should a machine
go down, its partner would take over and provide service.

VPC has an equivalent of a floating VIP - an ENI (Elastic Network Interface). ENIs can have one or more private IP addresses
(in the RFC1918 range) assigned to them, as well as an Elastic IP. ENIs can be attached and detached from machines.

## A solution

What I decided to do, and have been running successfully for about four months, is write a cluster resource agent that 
Pacemaker can use to move an ENI between machines. This ENI has a private IP address attached to it, which instances 
that require NAT routing can use as their default gateway. If one machine fails, the second machine can take over within
a minute by "stealing" the ENI from the failing machine.

## How it all works

This is going to get a bit complicated, so bear with me.

### IAM role

Firstly, you need to create an IAM role for your NAT cluster instances. IAM roles are assigned to an instance on creation,
and allow that instance to perform tasks against the AWS API without providing the usual access key and secret key.

This IAM role will allow your pair of NAT instances to attach and detach ENIs - this will be used by our Pacemaker 
resource agent to move the ENI holding the VIP between the instances in the cluster.
