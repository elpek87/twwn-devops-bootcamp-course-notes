# KUBERNETES ON AWS - EKS

## CONTAINER SERVICES ON AWS

EC2 - VMs in AWS
ECR - Docker image storage in AWS with multiple additional features (AWS specific)
ECS - AWS's own container orchestrator
EKS - Managed Kubernetes in AWS
FARGATE - Serverless containers in AWS

All of them integrate great.

Container Orchestration Tools = managing, scaling and deploying containers

Examples: Docker Swarm, Kubernetes, Mesos, Nomad, ECS

ECS characteristics:
- smiliar purpose to k8s - good for less complex apps
- ECS control plane is free
- AWS managed and specific to AWS
- simpler than EKS
- ECS makes control plane
- containers run on VMs that are managed by user (EC2) and communicate with Control Plane using ECS Agent
- gives full access to infrastructure but taking care of it is on user side

AWS Fargate + ECS - serverless way to launch containers that works with ECS:
- it provisions server resources on demand to run containers
- it always ensures that you have the exact infrastructure resources to run your containers
- you pay only for what you actually use (cpu, memory,  resources)
- easily scales without fixed resources defined beforehand
- you only care about the app you're deploying
- in some cases such reduced flexibility can be a problem

Both approaches are integrated with AWS ecosystem. ECS + EC2 + Fargate is also possible.

EKS - Elastic Kubernetes Service:
- provides K8S API
- open source and independent
- migration somewhere else is possible (other provider) but can be hard if you are integrated with AWS components heavily
- for the Worker Nodes - you create EC2 instances (Compute Fleet) and connect them to EKS
- Control Plane communicates with worker nodes using K8S processes (that you configure)
- EC2s are either self-managed here or semi-managed (EKS with Nodegroup)
-  Semi-managed approach makes creation, deletion EC2 instances automatic but you still need to configure them yourself to make them Worker Nodes in K8S
- Auto scaling is also not automatic.

When creating EKS Cluster AWS provisions K8S Control Plane Nodes with all services installed on them. By default these are replicated across Availability Zones. EKS also takes care of replicating, storing and backups of etcd.

AWS Fargate + EKS - serverless fully managed worker nodes for EKS

Mix of EC2 + EKS + Fargate is also possible.

EKS Setup:
1. Provision EKS Cluster with Control Plane Nodes
2. Provision Worker Nodes (Nodegroup of EC2s) or Fargate
3. Connect Nodegroup or Fargate to EKS cluster
4. Use your cluster with kubectl and deploy your app containers

## CREATE EKS CLUSTER WITH AWS MANAGEMENT CONSOLE

Steps to create EKS cluster:
1. EKS IAM Role
2. VPC for Worker Nodes (needed mainly for specific firewall groups)
3. EKS Cluster Control Plane Nodes
4. Connect to the cluster using kubectl

Steps to create Worker Nodes:
1. EC2 IAM Role For Node Group
2. Create Node Group to attach to EKS
3. Configure auto-scaling


EKS IAM Role:

- create IAM role in our AWS account
- Assign role to EKS cluster managed by AWS to allow AWS to create and manage components on our behalf

IAM:
- Trusted Entity Type: AWS Service
- Service or Use Case: EKS
- Use Case: EKS Cluster

VPC:
Best practice: configure public and private subnets

Instead of creating everything manually in VPC section we're using CloudFormation. AWS CloudFormation is an Infrastructure as Code (IaC) service that lets you define and provision AWS infrastructure using YAML or JSON templates.


## CONFIGURE AUTOSCALING IN EKS CLUSTER

## CREATE FARGATE PROFILE FOR EKS CLUSTER

## CREATE EKS CLUSTER WITH EKSCTL COMMANDLINE TOOL

## DEPLOY TO EKS CLUSTER FROM JENKINS PIPELINE

## BONUS: DEPLOY TO LKE CLUSTER FROM JENKINS PIPELINE

## JENKINS CREDENTIALS - NOTE ON BEST PRACTICES

## COMPLETE CI/CD PIPELINE WITH EKS AND DOCKERHUB

## COMPLETE CI/CD PIPELINE WITH EKS AND ECR
