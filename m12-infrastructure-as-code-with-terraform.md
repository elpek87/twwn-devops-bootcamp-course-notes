# INFRASTRUCTURE AS CODE WITH TERRAFORM

## INTRODUCTION TO TERRAFORM

Terraform is an Infrastructure as Code (IaC) tool created by HashiCorp.

It allows you to define, provision, and manage infrastructure using code instead of manually creating resources in cloud consoles.

Terraform is declarative - you define what end result you want unlike with imperative tools in which you define exact steps of how to achieve it.

Terraform vs Ansible - both IaC tools:

- Ansible is mainly a configuration tool that can configure infrastructure, deploy apps, install and update software

- Terraform is mainly infrastructure provisioning tool

What is Terraform good for?

- managing existing infrastructure (and automate continuous changes to it)
- replication infrastructure

Terraform architecture:

1. TERRAFORM CORE - it uses 2 input sources - tf-config amd state (current state of setup)

It takes input from these two input sources and then compares it with current state to plan what needs to be done to get to desired state (execution plan).

2. PROVIDERS - these are the plugins that allow Terraform to communicate with external APIs

They translate Terraform configuration into API calls, send the requests to cloud providers and return responses back to Terraform Core.

Declarative config file:

- adjust old config file and re-execute
- clean and small config file
- always know the current setup

Terraform stages:

1. REFRESH - refresh updates the state file to match the real-world infrastructure (by talking to providers)
2. PLAN - compares desired state with current state after refresh and produces execution plan
3. APPLY - executes the plan (changes) and updates state file
4. DESTROY - removes all resources defined in your configuration (or a targeted subset).

Terraform is an universal IaC tool.

## INSTALL TERRAFORM AND SETUP TERRAFORM PROJECT

For Mac just:

```
$ brew tap hashicorp/tap

$ brew install hashicorp/tap/terraform
```

## PROVIDERS IN TERRAFORM

In Terraform, a Provider is a plugin that allows Terraform to interact with external APIs.

There are a lot of providers that can be browsed here: https://registry.terraform.io/browse/providers

First you need to define provider either in your tf file or separate providers.tf:
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}
```

For the official providers it is not required to explicitly define the provider - it would still work without it because terraform would look in hashicorp registry.

Then execute:

`$ terffaform init`

It will fetch and install provider from the registry.


To connect to AWS using Terraform provider:
```
provider "aws" {
    region = "eu-central-1"
    access_key = ""
    secret_key = ""
}
```

Providers themselves exposes complete external API of the environment we're going to interact with.

## RESOURCES AND DATA SOURCES

Defining a resource is done as follows:

<provider>_<resourceType> <resource_custom_name>

For the resource that doesn't exist yet but need to be referenced we do (id):

```
resource "aws_vpc" "development-vpc" {
    cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "dev-subnet-1" {
    vpc_id = aws_vpc.development-vpc.id
}
```

For the above example if the AZ is not defined for the subnet Terraform will use the default one.

To apply changes defined in our tf file:
`$ terraform apply`

For the existing resource if we need to do some change we can query for its details using provider with the "data" keyword:

```
data "aws_vpc" "existing-vpc" {
    default = true
}

resource "aws_subnet" "dev-subnet-2" {
    vpc_id = data.aws_vpc.existing-vpc.id
    cidr_block = "172.31.32.0/20"
    availability_zone = "eu-central-1a"
}
```

PROVIDER = IMPORT LIBRARY

RESOURCE/DATA = FUNCTION CALL OF LIBRARY

ARGUMENTS = PARAMETERS OF FUNCTION

## CHANGE AND DESTROY TERRAFORM RESOURCES
When existing resource gets changed it is prefixed by yellow "~" when doing "apply". Adding something is prefixed by green "+" and removing something is prefixed by rd "-".

To add tag to a resource:

```
tags = {Name = "MyTag}
```

To remove a resource we can just remove it from our .tf file or using:

`$ terraform destroy -target <resource_type.resource_name>`

It's better to use tarraform apply way because destroy method leaves us with .tf file which does not reflect current stare of the infrastructure.

## TERRAFORM COMMANDS

Useful commands:
`$ terraform plan` - it's like a preview without applying the changes

`$ terraform aply -auto-approve` - immediate approval without needing to confirm

`$ terraform destroy` - if ran without parameters it would remove all of the resources in correct order. Often used to reverse changes with .tf file.

## TERRAFORM STATE

The most important files in Terraform are:

- terraform.tfstate (current state)
- terraform.tfstate.backup (previous state)

State management command:
`$ terraform state`

One of the most useful subcommands is:
`$ terraform state show <resource>`

It will give you values used to create certain resource etc. However it's not often used - managing the resources is done with .tf file itself.

## OUTPUT VALUES

Another way to see specific attribute of a resource is using output values:

```
output "dev-subnet-id" {
    value = aws_subnet.dev-subnet-1.id
}
```


## VARIABLES IN TERRAFORM

Variables in Terraform are used to make your configuration flexible, reusable, and easier to maintain. Instead of hard-coding values inside your Terraform files, you define variables and assign values to them when needed.

Variable is defined using "variable" keyword:
```
variable "subnet_cidr_block" {
    description = "subnet cidr block"
}
```

And then you reference it like this:

```
cidr_block = var.subnet_cidr_block
```

There are three ways to assign variable:

1. Using prompt when doing `$ terraform apply` - not the most comfortable way in the long run.
2. Using command line like: `$ terraform apply -var "subnet_cidr_block=10.0.10.0/24"`.
3. Using separate file `terraform.tfvars`.

When to use input variables?

When replicating same infrastructure for multiple environments.

In this case we can separate variables into multiple variables file like ***terraform-dev.tfvars***. The when using `$ terraform apply` an error would pop up because Terraform will not be able to find variables file. It needs to be passed as parameter using `$ terraform apply -var-file terraform-dev.tfvars`.

To define default value for variable in Terraform:

```
default = "10.0.10.0/24"
```

It's also possible to set type of a variable - useful for enforcing specific type for value:

`type = string` or `type = list(string)`

Then to access second element of the list - starts from 0:

```
cidr_block = var.cidr_blocks[1]
```

You can also use list of objects here:

```
 type = list(object({
        cidr_block = string
        name = string
    }))
```

And then reference them like this:

```
    cidr_block = var.cidr_blocks[0].cidr_block
    tags = {
        Name: var.cidr_blocks[0].name
    }
```

## ENVIRONMENT VARIABLES IN TERRAFORM

You can pass your provider credentials (or some other details) as environment variables like:

```
export AWS_SECRET_ACCESS_KEY=-xxx
export AWS_ACCESS_KEY_ID=yyy
```

Terraform would automatically pick them up when connecting to provider - here AWS. Terraform can also pick the credentials up from `~/.aws/credentials`.

You can also define your own global env vars with "TF_VAR" prefix - like that:

```
export TF_VAR_avail_zone="eu-central-1a"
```

Then you reference it in main terraform file like:

```
variable avail_zone{}

availability_zone = var.avail_zone
```

## CREATE GIT REPOSITORY FOR LOCAL TERRAFORM PROJECT

When working with Terraform not the whole project directory should be pushed into a repository - state files, .teraform/ dir and .tfvars (may contain sensitive data) should be excluded.

The .terraform.lock.hcl however can be stored in a repository so that all team members have the same version of the providers.

## AUTOMATE PROVISIONING EC2 WITH TERRAFORM - PART 1

When creating VPC in AWS some items like routing table and NACL get created by dafault. Keep in mind that newly created VPC doesn't route any traffic to/from the Internet - just internal VPC traffic. We need Internet Gateway to access the Internet.

Terraform doesn't really care about the order of the components defined in .tf file - it knows in which sequence they need to be created.

By default newly created subnets are not associated with newly created route tables. You can either associate them to the new route tables or just use the default one like this:

```
resource "aws_default_route_table" "main-rtb" {
    default_route_table_id = aws_vpc.myapp-vpc.default_route_table_id
    route {
        cidr_block = "0.0.0.0/0"
        gateway_id = aws_internet_gateway.myapp-igw.id
    }
    tags = {
        Name: "${var.env_prefix}-main-rtb"
    }
}

resource "aws_route_table_association" "a-rtb-subnet" {
    subnet_id = aws_subnet.myapp-subnet-1.id
    route_table_id = aws_route_table.myapp-route-table.id

}
```

Now we need to configure security group in AWS for incoming (ingress) and outgoing (egress) traffic:

It can also be done with the default SG.

```
resource "aws_security_group" "myapp-sg" {
    name = "myapp-sg"
    vpce_id = aws_vpc.myapp-vpc.id

    ingress {
        from_port = 22
        to_port = 22
        protocol = "TCP"
        cidr_block = [var.my_ip]
    }

    ingress {
        from_port = 8080
        to_port = 8080
        protocol = "TCP"
        cidr_block = ["0.0.0.0/0"]
    }

    egress {
        from_port = 0
        to_port = 0
        protocol = "-1"
        cidr_block = ["0.09.0.0/0"]
        prefix_list_ids = []
    }
}
```



Comments in tf files can be done with:

```
 /* */
```

## AUTOMATE PROVISIONING EC2 WITH TERRAFORM - PART 2

To get the latest version of Amazon Linux image that is available (programmatically) just use this and filter out what's needed:

```
data "aws_ami" "latest-amazon-linux-image" {
    most_recent = true
    owners = ["amazon"]
    filter {
        name = "name"
        values = ["al2023-ami-*-kernel-*-arm64"]
    }
    filter {
        name = "virtualization-type"
        values = ["hvm"]
    }
}
```
When creating an instance you can specify its SSH key pair in Amazon:

```
resource "aws_instance" "myapp-server" {
    ami = data.aws_ami.latest-amazon-linux-image.id
    instance_type = var.instance_type

    subnet_id = aws_subnet.myapp-subnet-1.id
    vpc_security_group_ids = [aws_default_security_group.default-sg.id]
    availability_zone = var.avail_zone

    associate_public_ip_address = true
    key_name = "ec2-server-key"

    tags = {
        Name: "${var.env_prefix}-server"
    }
}
```

SSH key pair can also be copied over to the newly created EC2 instance. There's one thing to remember - the key pair itself must be generated locally.

Then you can reference in two ways:

```
resource "aws_key_pair" "ssh-key" {
    key_name = "server-key"
    public_key = var.my_public_key
}
```

Or using file location - here in variable:

```
resource "aws_key_pair" "ssh-key" {
    key_name = "server-key"
    public_key = file(var.public_key_location)
}
```

## AUTOMATE PROVISIONING EC2 WITH TERRAFORM - PART 3

To make Terraform execute commands on created instance we use (in instance block):

```
 user_data = <<-EOF
    #!/bin/bash
    dnf update -y
    dnf install -y docker
    systemctl enable docker
    systemctl start docker
    docker run -d -p 8080:80 nginx
    EOF
    user_data_replace_on_change = true # start fresh when instance is recreated
```

Or using a script and referencing it with file:

```
user_data = file("entry-script.sh")
```

Terraform is for infrastructure management - it's not the best tool to manage applications.



## PROVISIONERS IN TERRAFORM

## MODULES IN TERRAFORM - PART 1

## MODULES IN TERRAFORM - PART 2

## MODULES IN TERRAFORM - PART 3

## AUTOMATE PROVISIONING EKS CLUSTER WITH TERRAFORM - PART 1

## AUTOMATE PROVISIONING EKS CLUSTER WITH TERRAFORM - PART 2

## AUTOMATE PROVISIONING EKS CLUSTER WITH TERRAFORM - PART 3

## COMPLETE CI/CD WITH TERRAFORM - PART 1

## COMPLETE CI/CD WITH TERRAFORM - PART 2

## COMPLETE CI/CD WITH TERRAFORM - PART 3

## REMOTE STATE IN TERRAFORM

## TERRAFORM BEST PRACTICES

