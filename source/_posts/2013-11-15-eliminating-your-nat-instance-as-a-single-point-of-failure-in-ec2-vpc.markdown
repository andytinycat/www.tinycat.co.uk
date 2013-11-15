---
layout: post
title: "Eliminating your NAT instance as a single point of failure in EC2 VPC"
date: 2013-11-15 19:51
comments: true
categories: sysadmin ec2
---

[Amazon's VPC](aws.amazon.com/vpc/) is wonderful, allowing you to push a section of your internal network into 
the cloud, with full control over routing, addressing, subnetting, etc. 

There's one obvious sore point, which is that by default, VPC machines do not get a public IP address like their EC2 Classic
counterparts. The upshot of this is that they cannot access the Internet without what Amazon refer to as a 
[NAT instance](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_NAT_Instance.html).

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

## How it all works: Part 1, creating the instances/ENI

This is going to get a bit complicated, so bear with me.

### IAM role

Firstly, you need to create an IAM role for your NAT cluster instances. IAM roles are assigned to an instance on creation,
and allow that instance to perform tasks against the AWS API without providing the usual access key and secret key.

This IAM role will allow your pair of NAT instances to attach and detach ENIs - this will be used by our Pacemaker 
resource agent to move the ENI holding the VIP between the instances in the cluster.

Log into you AWS console, and go to IAM. Click the *Create New Role* button, give it a descriptive name, choose the type
of role to be *EC2*, choose *Custom Policy*, then paste the following into the textarea:

    {
      "Statement": [
        {
          "Sid": "Stmt1369471163971",
          "Action": [
            "ec2:AttachNetworkInterface",
            "ec2:DetachNetworkInterface",
            "ec2:DescribeNetworkInterfaces"
          ],
          "Effect": "Allow",
          "Resource": [
            "*"
          ]
        }
      ]
    }

### Create two NAT instances

Create two NAT instances launched into your VPC, in the same subnet, named something descriptive like `nat-a` and `nat-b`. 
After they launch, select them in the EC2 console, right-click and select *Change Source/Dest Check*. Click the *Yes, Disable* button. 
Because these instances will be routing traffic for other machines, failure to disable this check will cause EC2 to drop
routed traffic from this machine, and leave you scratching your head in confusion.

### Create the NAT security group

Create a security group in VPC called "nat", and allow all traffic from whatever instances will be using NAT (I have every instance
in 'default' among one or more other groups, so I just permit access from 'default').

### Create an ENI

In the left menu of the EC2 console, go to *Network Interfaces*. Hit *Create Network Interface*, give it a description, 
choose the subnet that your two NAT instances are already in, and then choose an IP address inside that subnet. Select "nat"
as the security group for this interface.
