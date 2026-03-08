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

## VARIABLES IN TERRAFORM

## ENVIRONMENT VARIABLES IN TERRAFORM

## CREATE GIT REPOSITORY FOR LOCAL TERRAFORM PROJECT

## AUTOMATE PROVISIONING EC2 WITH TERRAFORM - PART 1

## AUTOMATE PROVISIONING EC2 WITH TERRAFORM - PART 2

## AUTOMATE PROVISIONING EC2 WITH TERRAFORM - PART 3

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

