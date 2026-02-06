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

Launching an EC2 instance is pretty straightforward - selecting OS, resources, login key pair and adjusting network configuration and security groups.

Docker image to run on the server is in the repo:

https://github.com/techworld-with-nana/react-nodejs-example

1. Cloned the repo. Fixed the Dockerfile since version 10 of node image would not compile the code.

2. Built it with:

```
$ docker buildx build --platform linux/arm64,linux/amd64 -t demo-app:1.0 .
```
3. Tagged the image with my repo:

```
$ docker tag demo-app:1.0 elpek87/demo-app:1.0
```
4. Logged into Dockerhub with:

```
$ docker login
```

5. Pushed the image:

```
$ docker push elpek87/demo-app:1.0
```

On the server:

1. Installed docker:

```
# dnf install docker && systemctl start docker
```

2. Pulled the image from Dockerhub and ran it:

```
docker run -d -p 3000:3080 elpek87/demo-app:1.0
```

After adjusting security groups to allow inbound traffic on port 3000 the app was accessible:

```
curl -I http://3.66.215.226:3000
HTTP/1.1 200 OK
```

## DEPLOY TO EC2 SERVER FROM JENKINS PIPELINE CI/CD

To deploy to EC2 server using Jenkins pipeline first we need the SSH Agent plugin installed on Jenkins.

Jenkins is going to use SSH private key from AWS to SSH onto EC2 in AWS.

1. First we need to add (pipeline scoped) credentials of type "SSH username with private key" to credentials store in Jenkins.
2. Next step is to add "SSH Agent" step in Pipeline Syntax under aws-multibranch-pipeline.
3. Hit "Generate Pipeline Script" which produces for Jenkinsfile:
```
sshagent(['ec2-server-key']) {
    // some block
}
```
Host key on EC2 may change so we'll use `-o StrictHostKeyChecking=no` in ssh command definition in Jenkinsfile.

Deploying single container by calling docker run on remote machine is alright but when the project needs multiple containers best way to deploy it is to use docker compose file.

First we need compose plugin on AmazonLinux:
```
# curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 \
  -o /usr/libexec/docker/cli-plugins/docker-compose

# chmod +x /usr/libexec/docker/cli-plugins/docker-compose
```

When using Docker Desktop this step can be skipped - compose plugin is installed out of the box.

## ECR - Elastic Container Registry

Amazon Elastic Container Registry (ECR) is a fully managed Docker container image registry. Think of it as AWS’s private Docker Hub, but tightly integrated with the rest of AWS.

First we just create the repository (private).

In ECR you create repository per image - not a repository where you can store multiple images.

To push the image to ECR registry first you have to login to AWS with awscli:

```
$ awscli login
```
Then the image needs to be tagged accordingly (renamed):

```
$ docker tag my-js-app:1.0 438987839758.dkr.ecr.eu-central-1.amazonaws.com/my-js-app:1.0
```

## Introduction to AWS CLI

AWS CLI (Command Line Interface) is a tool that lets you control AWS from your terminal instead of clicking around in the web console.

You can:
- Provision infrastructure
- Deploy apps
- Rotate credentials
- Clean up resources

…all inside:
- Bash scripts
- CI/CD pipelines (Jenkins, GitHub Actions, GitLab CI)
- Makefiles

Initial configurations is done with:

`$ aws configure`

Configurations is stored in:

`~/.aws`

# Basic AWS CLI operations:

Info on configured security groups:
`$ aws ec2 describe-security-grpups`

Info on VPCs:
`$ aws ec2 describe-vpcs`

Creating security groups:
`$ aws ec2 create-security-group --group-name my-sg --description "My SG" --vpc-id vpc-xxx - creates security group`

Allow incoming traffic on port 22 from specific IP:
`$ aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 22 --cidr IP/MASK`

Creating SSH key pair and saving private key:
`$ ❯ aws ec2 create-key-pair --key-name MyKeyCli --query 'KeyMaterial' --output text > mykpcli.pem`

Creating EC2 instance:
```
$ aws ec2 run-instances \
--image-id ami-0191d47ba10441f0b \
--count 1 \
--instance-type t2.micro \
--key-name MyKeyCli \
--security-group-ids sg-xxx \
--subnet-id subnet-xxx
```
Display information on EC2 instances:
`$ aws ec2 describe-instances`

All of the commands output can also be filtered - filter and query.

To display InstanceIDs of EC2s of t2.micro type:
`$ aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro" --query "Reservations[].Instances[].InstanceId"`

To display instances running specific AMI version:
`$ aws ec2 describe-instances --filters "Name=image-id,Values=ami-0191d47ba10441f0b"`

# IAM CLI Commands

Creating a group:
`$ aws iam create-group --group-name MyGroupCli`

ARN stands for Amazon Resource Name. It’s a globally unique identifier for any AWS resource.

Creating a user:
`$ aws iam create-user --user-name MyUserCli`

Adding user to a group:
`$ aws iam add-user-to-group --user-name MyUserCli --group-name MyGroupCli`

Display users in group:
`$ aws iam get-group --group-name MyGroupCli`

Getting policy ARN:
```
$ aws iam list-policies --query 'Policies[?PolicyName==`AmazonEC2FullAccess`].Arn' --output text
```

Attach policy to a group:
`$ aws iam attach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess`

List groups attached to a policy:
`$ aws iam list-attached-group-policies --group-name MyGroupCli`

Define password for user:
`$ aws iam create-login-profile --user-name MyUserCli --password xyz --password-reset-required`

Describe user:
`$ aws iam get-group --group-name MyGroupCli`

Creating custom policy - for example to allow user to change password requires JSON file:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:ChangePassword",
      "Resource": "arn:aws:iam::*:user/MyUserCli"
    },
    {
      "Effect": "Allow",
      "Action": "iam:GetAccountPasswordPolicy",
      "Resource": "*"
    }
  ]
}
```

Then you define it from file:
`$ aws iam create-policy --policy-name changePwd --policy-document file://changePasswordPolicy.json`

Attaching policy to a group:
`$ aws iam attach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::*:policy/changePwd`

Creating Access Keys for a new user:
`aws iam create-access-key --user-name MyUserCli`

Switch user in AWSCLI:
`$ aws configure` <= that would override the defaults

`$ aws configure set aws_access_key_id ID`
`$ aws configure set aws_aws_secret_access SECRET` <= that also messes the defaults

To keep the defaults:
`$ export AWS_ACCESS_KEY=ACCESS`
`$ export AWS_SECRET_ACCESS_KEY=SECRET`
