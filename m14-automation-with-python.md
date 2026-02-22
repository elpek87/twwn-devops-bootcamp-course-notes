# AUTOMATION WITH PYTHON

## INTRODUCTION TO BOTO LIBRARY (AWS SDK FOR PYTHON)

Boto is the official Python SDK for AWS (Amazon Web Services). Instead of clicking in the AWS Console, you write Python code to manage infrastructure.

## INSTALL BOTO3 AND CONNECT TO AWS

Boto can be installed from pip:
`$ pip install boto3`

Boto uses credentials from ~/.aws directory - if you have them configured for awscli - there's no need to do anything in that matter.

## GETTING FAMILIAR WITH BOTO

Named parameters:
- pass arguments via key = value
- sometimes called "keyword arguments"
- this way the order of parameters doesn't matter and you don't have to pass them in the order that function requires

Client vs Resource:

Client = more low level API

Resource = high-level and object-oriented

Resource gives us object we can use for subsequent calls.

## TERRAFORM VS PYTHON - UNDERSTAND WHEN TO USE WHICH TOOL

Terraform manages state of the infrastructure:

- knows the current state
- knows the difference between current state and desired state
- is idempotent (operation can be applied multiple times without changing the end result)
- you declare the end result (by either adding or removing code)
- is more high level

Python in this case:

- doesn't have a state
- is not idempotent
- needs explicitly written code to delete resources
- is more low level (boto)

Use case for Boto3:

- you can do much mode - thanks to its low level API
- more complex logic is possible since python is full programming language

## HEALTH CHECK: EC2 STATUS CHECK

## WRITE A SCHEDULED TASK IN PYTHON

## CONFIGURE SERVER: ADD ENVIRONMENT TAGS TO EC2 INSTANCES

## EKS CLUSTER INFORMATION

## BACKUP EC2 VOLUMES: AUTOMATE CREATING SNAPSHOTS

## AUTOMATE CLEANUP OF OLDS SNAPSHOTS

## AUTOMATE RESTORING EC2 VOLUME FROM BACKUP

## HANDLING ERRORS

## WEBSITE MONITORING 1: SCHEDULED TASK TO MONITOR APPLICATION HEALTH

## WEBSITE MONITORING 2: RESTART APPLICATION AND REBOOT SERVER
