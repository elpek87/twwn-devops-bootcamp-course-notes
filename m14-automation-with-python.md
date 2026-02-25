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

Boto provides simple way to do a health check on our running EC2 instances - to do so we use two functions:

- describe_instances()
- describe_instance_status()

## WRITE A SCHEDULED TASK IN PYTHON

State of the EC2 instances change so it's useful to have a script running with some time schedule. Python allows that using *schedule* library.

To run function every 5 minutes just do:

```
schedule.every(5).minutes.do(check_instance_status)
```

The library itself is pretty powerful - does everything and more what crons can do.

## CONFIGURE SERVER: ADD ENVIRONMENT TAGS TO EC2 INSTANCES

Keep in mind that such tagging shouldn't be done one by one - tag them all with single HTTP request.

## EKS CLUSTER INFORMATION

Getting information on EKS cluster is also done with boto3.client module.

## BACKUP EC2 VOLUMES: AUTOMATE CREATING SNAPSHOTS

Volumes are AWS Storage Components that attach to EC2 servers.

Volume Snapshot = copy of Volume.

To keep data persistent EC2s should be stopped during the snapshot.

## AUTOMATE CLEANUP OF OLD SNAPSHOTS

Amazon creates snapshots in the background - we want to operate on snapshots created by us - that's why we should use filters.

To sort snapshots by "StartTime" we use sorted function as follows:
```
sorted(snapshots, key=itemgetter('StartTime'))
```

itemgetter('StartTime') creates a function that:
- takes one element (e.g., a dictionary)
- returns element['StartTime']

Sorted come from operator library (build-in).

## AUTOMATE RESTORING EC2 VOLUME FROM BACKUP

Volume created from snapshot need to be in the same AZ as the EC2 instance itself.

Attach volume to a different device thant the one EC2 currently uses as primary.

## HANDLING ERRORS

## WEBSITE MONITORING 1: SCHEDULED TASK TO MONITOR APPLICATION HEALTH

## WEBSITE MONITORING 2: RESTART APPLICATION AND REBOOT SERVER
