# AWS Services

## INTRODUCTION TO AWS

Amazon Web Services (AWS) is Amazon’s cloud computing platform.

In simple terms: it lets people and companies rent computing power, storage, databases, and other tech services over the internet, instead of owning and running physical servers. You pay only for what you use, and it scales up or down as needed.

Scopes of services:

- Global (Users, permissions, billing)
- Region (S3, VPC, DynamoDB)
- AZ (EC2, EBS, RDS)

## IAM - MANAGE USERS, ROLES AND PERMISSIONS

Amazon IAM (short for Identity and Access Management) is a security service from Amazon Web Services that controls who can access your AWS resources and what they’re allowed to do.

IAM does:

1. Authentication - identity verification - users, roles, federated users
2. Authorization - permissions

Least Privilege Principle is followed.

Core of IAM:

- users (root is default)
- groups
- roles
- policies

It is advised to create admin user instead of using the default root.

Permissions - what actions are allowed (live in policy)

Policies - definition of what is allowed, where and under what conditions

Groups - permissions "bundle" for users

Roles - temporary identity with permissions (to a service for example)

## PROGRAMMATIC ACCESS

For programmatic access generating access key is required.

## REGIONS AND AVAILABILITY ZONES

An AWS Region is a distinct geographic location that hosts multiple AWS data centers, allowing you to run applications close to users and meet reliability or compliance needs. There are 30+ regions.

An Availability Zone is a separate data center (or group of data centers) inside an AWS Region that helps keep applications running even if part of the infrastructure fails.

## VPC - MANAGE PRIVATE NETWORK ON AWS

In AWS, a VPC (Virtual Private Cloud) is your own logically isolated network in the cloud where you can launch and control AWS resources as if they were in a traditional data center.

- VPC is regional
- VPC spans all the AZs in that Region
- VPC is private, isolated network

Subnets:

A subnet is a smaller IP network (CIDR block) carved out of a VPC, and it exists in one specific Availability Zone. There can be private and public subnets (depends on firewall configuration).

VPC assigns you the IP addresses block that can be used as internal IP range on VPC level - internal communication only. Each subnet gets IP range from that IP block.

An Internet Gateway is a VPC component that enables internet connectivity for resources in public subnets by providing routing and public IP translation.

Network ACLs (NACLs) are stateless, subnet-level firewalls.

Security Groups (SGs) are stateful, instance-level firewalls.

## CIDR BLOCKS EXPLAINED

A CIDR block (Classless Inter-Domain Routing) defines a range of IP addresses using a compact notation.

http://jodies.de/ipcalc?host=10.0.0.0&mask1=1&mask2=

http://www.davidc.net/sites/default/subnets/subnets.html

## INTRODUCTION TO EC2 VIRTUAL CLOUD SERVER

Amazon EC2 stands for Amazon Elastic Compute Cloud. It’s a core service from Amazon Web Services (AWS) that lets you rent virtual computers in the cloud and run applications on them—without owning or maintaining physical servers.
