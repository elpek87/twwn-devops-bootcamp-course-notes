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

Main reason for creating VPC for EKS are Worker Nodes and firewall between them and Control Planes (these are on AWS service account, not mine)

Instead of creating everything manually in VPC section we're using CloudFormation. AWS CloudFormation is an Infrastructure as Code (IaC) service that lets you define and provision AWS infrastructure using YAML or JSON templates.

When setting up an EKS cluster you can select EKS Auto Mode which is a feature of Amazon EKS that automatically manages the worker infrastructure for your Kubernetes cluster — including provisioning, scaling, and maintaining nodes — so you don’t have to manage EC2 instances, node groups, or autoscaling groups yourself.

In "Cluster access" section select "EKS API and ConfigMap".

In Networking configuration it is important to select VPC stack that we created before using CloudFormation - pay attention to VPC selection, subnets and security group.

In Amazon EKS, Cluster endpoint access controls how the Kubernetes API server is reachable — meaning how kubectl, CI/CD pipelines, or automation tools connect to your cluster.

This is about network access to the control plane, not your worker nodes or services.

Optimal choice here is "Public and private" which gives good security, API inside VPC and ability to use kubectl from laptop.

In "Select add-ons" section we generally leave the defaults selected - the most important ones are:
- Amazon VPC CNI - handles pod networking, assigns real IPs to pods
- CoreDNS - provides DNS inside the cluster
- kube-proxy - handles service networking, routes traffic to correct pods and manages iptables rules
- EBS CSI driver - enables dynamic provisioning of PersistentVolumes and EBS disks
- EFS CSI Driver - shared storage for stateful workloads
- AWS Load Balancer Controller - creates application and network load balancers

To add nodes to an EKS cluster go to "Compute" section in cluster dashboard and then depending on whether you choose EC2s or Fargate you start there. It's advised and convenient to use "Node groups" but that requires IAM EC2 Role for it to work. Basically the role is for Kubelet to be able to perform actions on Node group.

In IAM -> Create Role -> Use case: EC2

- AmazonEKSWorkerNodePolicy
- AmazonEC2ContainerRegistryReadOnly (to allow to download images from image registry)
- AmazonEKS_CNI_Policy (networking stuff)

When creating Node Group:
- select AMI type
- capacity type
- instance type
- scaling configuration (3 nodes, min 2 nodes, max 4 nodes) - can be done with autoscaler.

After creating a node group EC2s are going to be provisioned automatically and come with containerd, kubelet and k-proxy preinstalled.

## CONFIGURE AUTOSCALING IN EKS CLUSTER

Cluster Autoscaler (CA) automatically adjusts the number of worker nodes in your cluster. It adds EC2 instances when pods can’t be scheduled or removes EC2 instances when they are underutilized.

An Auto Scaling Group (ASG) is an AWS service that automatically creates, replaces, and removes EC2 instances. ASG maintains:

- Desired capacity
- Minimum size
- Maximum size

It's good to set maximum capacity for:

- cost control (traffic spikes)
- budget
- protection (indication of security or performance issues)

Minimum capacity is also good to have because:

- baseline level of availability, performance and reliability
- fault tolerance
- scaling delays avoidance

Cluster Autoscaler in k8s will run as service account.

Autoscaler is k8s component but needs to be able to do scaling on level of AWS (EC2) to scale the cluster. IAM roles works only between AWS services. For *external* services we're going tu use OIDC = identity layer on to op OAuth 2.0.

To configure trust between them - Configure OIDC provider in AWS - register EKS OIDC Provider URL as a trusted identity provider in IAM

Based on that trust we create IAM Role that allows EC2 creation / deletion . this role is configured with *Web Identity* that points to OIDC provider.

Workflow:

1. Autoscaler sends request to assume Role
2. AWS verifies token is issued by EKS OIDC provider
3. As an authenticated identity Autoscaler can assume Role
4. With temporary credential - Autoscaler has permission to interact with AWS API and handle EC2

That's secure - specific trust relationship - specific k8s components interact with specific AWS resources.

Autoscaling group -> custom policy and attach to IAM Role -> deploy autoscaler

You need to add identity provider in IAM using OpenID provider URL of the cluster. Audience must be set to **sts.amazonaws.com** (Security Token Service) - it ensures that token is used only for authentication purpose and can be used to assume roles.

Then we go to "Roles" and create Trusted entity type of **Web identity** then select OIDC in Web identity drop-down list and the audience of sts.amazonaws.com as well. In permissions select "ClusterAutoscalerPolicy" give the role a name "EKSService Account Role"

Now in EC2 Auto scaling group we go to "Tags" - we need them because K8S will use them to auto discover Auto scaling groups in AWS.

These are the important tags:
- k8s.io/cluster-autoscaler/eks-cluster-test
- k8s.io/cluster-autoscaler/enabled

In most cases the tag part is automatically configured.

K8S Autoscaler config - we use this file:
https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

Minor edits are:

1. Service account - arn is from IAM EKSServiceAccount Role:

```
annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam:ACCOUNTID:role/EKSServiceAccountRole
```

2. Make sure autoscaler won't evict itself when scaling down the nodes:

```
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
        cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
```
3. Modify the autoscaler command itself:
```
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<YOUR CLUSTER NAME>
            - --balance-similar-node-groups
            - --skip-nodes-with-system-pods=false
```

4. Make sure autoscaler image version matches k8s version:

```
    - image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.35.1
```

5. Set AWS region for the autoscaler:

```
          name: cluster-autoscaler
          env:
            - name: AWS_REGION
              value: "eu-central-1"
```

Verify with:
`$ kubectl get deployment -n kube-system cluster-autoscaler`

AWS auto scaler feature is not that fast - it takes time for it to detect that the cluster needs to scale and then some more time to provision more EC2s.

At this point if you bump up deployment replicas for nginx 1 -> 20 autoscaler should start additional nodes to adjust.

## CREATE FARGATE PROFILE FOR EKS CLUSTER

In EKS you have two main ways to run Pods:

- Node Groups = you run Kubernetes Pods on EC2 worker nodes
- Fargate Profiles = AWS runs the worker nodes for you (serverless Pods)

Fargate provisions VM per POD.

When using Fargate there are some limitations:

- no support for stateful applications yet
- no support for DaemonSets yet

1. IAM Role for Fargate

Trusted entity: AWS Service
Use case: EKS -> EKS - Fargate POD

2. Fargate profile under EKS / Compute

Fargate profile is basically a mapping rule that decides which pods should run on serverless infrastructure (Fargate) rather than on EC2 nodes.

Fargate needs namespace selectors for the PODs. There's also an option to configure label selector.

These are useful in scenarios in which for example we have DEV and TEST environment in the same cluster and we want to run namespace "dev" through Fargate. Or for example run stateful apps using Node Groups (because of Fargate limitations).

EC2 and Fargate nodes are prefixed different:
```
❯ kubectl get nodes -n dev
NAME                                                       STATUS   ROLES    AGE   VERSION
fargate-ip-192-168-165-227.eu-central-1.compute.internal   Ready    <none>   64s   v1.35.0-eks-70ce843
ip-192-168-155-133.eu-central-1.compute.internal           Ready    <none>   36m   v1.35.0-eks-70ce843
```

## CREATE EKS CLUSTER WITH EKSCTL COMMANDLINE TOOL

eksctl is a command-line tool used to create and manage Kubernetes clusters on Amazon Web Services (AWS) using Amazon Elastic Kubernetes Service (EKS). It is the official CLI tool for EKS, created by Weaveworks.

If awscli was configured before eksctl should work out of the box using same credentials as the ones listed:

`$ aws configure list`

If there's no need for cluster configuration to be changed and defaults are ok then running this will get your cluster up:

`$ eksctl create cluster`

It also takes parameters:

```
eksctl create cluster --name demo-cluster \
--version 1.35 \
--region eu-central-1 \
--nodegroup-name demo-nodes \
--node-type t2.micro \
--nodes 2 \
--nodes-min 1 \
--nodes-max 3
```

Eksctl also allows you to generate yaml file which can be passed to EKS control.

## DEPLOY TO EKS CLUSTER FROM JENKINS PIPELINE

To deploy to an EKS cluster from Jenkins we need to:

1. Install kubectl tool inside jenkins container

First access the jenkins container as root user: `$ docker exec -u 0 -it epic_stonebraker bash`

Then execute this from inside the container:

`# curl -LO https://dl.k8s.io/release/v1.35.0/bin/linux/arm64/kubectl; chmod +x ./kubectl; mv ./kubectl /usr/local/bin/kubectl`

2. Install aws-iam-authenticator inside jenkins container (authentication)

`# curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.7.11/aws-iam-authenticator_0.7.11_linux_arm64; chmod +x ./aws-iam-authenticator; mv aws-iam-authenticator /usr/local/bin`

3. Create kubeconfig to connect to EKS cluster

Config should be adjusted from the tamplate and placed in $JENKINS_HOME/.kube in docker container:

```
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: <certificate-data>
    server: <endpoint-url>
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: aws
  name: aws
current-context: aws
users:
- name: aws
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: /usr/local/bin/aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - <cluster-name>
```

4. Add AWS credentials on Jenkins fo AWS account authentication

We need to add - type "Secret text" in multibranch pipeline scoped credentials store:

- aws_access_key_id
- aws_secret_access_key

Both are to be found in ~/.aws/credentials

then we need to reference them in Jenkinsfile:
```
AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
```

## BONUS: DEPLOY TO LKE CLUSTER FROM JENKINS PIPELINE

There are different approaches in deployments to k8s that do not involve platform specific mechanisms like we have in AWS. It's similar to interacting with bare metal k8s cluster. Deploying to Akamai (Linode) involves:

This may also be necessary:
`$ export KUBECONFIG=~/.test-kubeconfig.yaml`

0. Configuring credentials in Jenkins

This time of "Secret file" kind was used.

1. Installing Kubernetes CLI Jenkins plugin (executing kubectl with kubeconfig credentials)

Plugin is called "Kubernetes CLI".

2. Configure Jenkinsfile to deploy to LKE cluster

It's all about defining credentials for the LKE with:
```
withKubeConfig([credentialsId: 'lke-credentials', serverUrl:'https://...'])
```

## JENKINS CREDENTIALS - NOTE ON BEST PRACTICES

When using SSH keys authentication: it's recommended to create a jenkins user on remote vm such as ec2 and configure ssh keys for that user to use in jenkins as credentials.

When using kubeconfig.yaml (such as Akamai LKE): it's recommended to create jenkins user in k8s (jenkins service account) and give it only required permissions and then create user token for the user.

## COMPLETE CI/CD PIPELINE WITH EKS AND DOCKERHUB

## COMPLETE CI/CD PIPELINE WITH EKS AND ECR
