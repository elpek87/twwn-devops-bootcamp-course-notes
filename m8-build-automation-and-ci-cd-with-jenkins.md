![Logo](assets/devops-bootcamp-logo.png)

## BUILD AUTOMATION & CI/CD WITH JENKINS

Build automation is a practice of automatically turning source code into a working, testable, and deployable artifact—without manual steps.

Whole process is possible thanks to build automation tools that automate tasks like compiling code, resolving dependencies, running tests, packaging artifacts, and publishing outputs — without manual intervention.

Jenkins is an open-source server that automates building, testing, and deploying software whenever something happens—like a Git commit or a scheduled trigger. It doesn’t compile code by itself but orchestrates the tools that do.

What needs to be configured?

Run test -> build tools
Build application -> build tools and/or Docker
Publish artifact -> store credentials to authenticate with repository

## Install Jenkins

We're going to use an official image of Jenkins from DockerHub:

`$ docker run -d -p 8080:8080 -p 5000:5000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts`

Port 8080 is used as listening port for Jenkins and port 5000 is used for agents communication with Jenkins server.

Initial admin password can be found in the named volume of Jenkins (or accessed from inside of docker container):

`$ cat /var/lib/docker/volumes/jenkins_home/_data/secrets/initialAdminPassword`

There are two ways of installing tools needed by Jenkins - either directly on the server or using Jenkins Plugins.

To install tools directly in jenkins docker container you need to access root account there:

`docker exec -u 0 -it cf4d40eb01a /bin/bash`

Official Jenkins container is built based on Debian - we can use a script to install node repo configuration but first we need to fetch it from the Internet to our container. We need curl or wget:

`# apt install curl`

Then download the script:

`# curl -sL https://deb.nodesource.com/setup_20.x -o nodesource_setup.sh`

Make it executable:

`# chmod +x nodesource_setup.sh`

Run it:

`# ./odesource_setup.sh`

Now we can intall node:

`# apt install nodejs`

Now NodeJS is available in Jenkins.

There's a very useful plugin "Stage View" - install it.

## Jenkins Basics Demo - Freestyle Job

Jenkins has multiple job types - we're starting with "Freestyle project". A Freestyle project in Jenkins is the most basic and flexible job type you can create in Jenkins. Think of it as a blank canvas where you manually define what steps Jenkins should run.

To automate build process section "Build Steps" should be used. In this case to interact with npm we can use "Execute shell commands" (since npmTo use  is installed in container) and maven is here as a plugin so we select "Invoke top-level Maven targets".

To see results of build once it finishes use "Console Output".

To use git repository it needs to be configured under "Source Code Management" in job configuration. It allows to configure git credentials as well. If a job is connected to git repository Jenkins checks out the code and if it needs to do something with the code it does it **locally**.

For testing purposes there is a script called freestyle-build.sh available on jenkins-jobs branch - if we set up this branch we're now able to run this script in Build Steps -> Execute shell. We run it like this:

`sh freestyle-build.sh`


The other job is maven:

1. Configure git repository - https://gitlab.com/twn-devops-bootcamp/latest/08-jenkins/java-maven-app - and branch: jenkins-jobs.
2. In Build Steps specify "test" command and then "package" command.

Results of the job 0 i.e. artifacts are stored in:

`jenkins_home/workspace/job_name/target`

## Docker in Jenkins

To make docker available in our docker container that runs Jenkins we can mount docker runtime directory with its socket from the host to the container.

This is just for this module - **never do it on production servers** - breaks isolation and you're giving root powers on the host to the container.

To finish docker installation inside the container you need to access it as root (-u 0) and run:

`# curl https://get.docker.com/ > dockerinstall && chmod 777 dockerinstall && ./dockerinstall`

And adjust permissions on socket:

`# chmod 666 /var/run/docker.sock`

Yet again remember that you're messing with socket permissions on the host here.

## Building Docker Image

In jobs build steps execute shell command of:

`docker build -t java-maven-app:1.0 .`

after the step that runs:

`package` for maven.

To push an image to the docker repository (DockerHub) - in this case DockerHub private repo in build step you need to tag the image with your repo:

`docker build -t elpek87/demo-app:1.0 .`
`echo $PASSWORD | docker login -u $USERNAME --password-stdin`
`docker push elpek87/demo-app:1.0`

For docker login we use bindings for username and password so that we don't have to provide credentials in command - Jenkins has them stored in its credentials store. In this case we don't have to specify target server - Dockerhub is the default one. Password is passed as piped to next command stdin.

Now let's push it to Nexus. First we need to configure our docker that runs in our Jenkins environment to allow insecure repositories. To do that on docker host edit /etc/docker/daemon.json and define:

```
{
	"insecure-registries":["NEXUS-IP:8083"]
}
```
# Freestyle to Pipeline Job

Chained Freestyle Jobs have their limitations - mostly to the UI and what plugins offer. Something more scriptable was needed, much better for CI/CD scenarios and that's why Pipeline Jobs were introduced in Jenkins.

it’s a scripted workflow that tells Jenkins exactly how your app should be built and released, step by step. Pipelines are scripted in Groovy.

There's a checkbox in Pipeline definition section - "Use Groovy Sandbox" - that protects Jenkins from potentially dangerous code. Block unsafe methods, runs the script in restricted environment etc. If unchecked then such script needs to be reviewed and approved by Jenkins administrator.

Best practice in IaC is to keep your pipeline scripts together with the app in git repository - then in jenkins we use "Pipeline script from SCM".

Jenkins pipelines can be written scripted or declarative.

Scripted:

- initial syntax
- Groovy engine
- flexible and powerful yet quite complex

Declarative:
- recent addition to pipelines
- less powerful but easier to get started
- predefined structure

Jenkinsfile requirements:

1. "pipeline" - top level
2. "agent" - where to execute (useful if jenkins is clustered)
3. "stages" - where all the work happens
   - it uses "stage" and "steps"

Pipeline jobs allow you to:

- run multiple tasks in parallel
- take user input
- evaluate conditional statements

All of the above is not easily done using plugins (chained freestyle) not to mention maintenance overhead when using them!

# Jenkinsfile Syntax


## POST

Post actions (post) - run steps after a stage or pipeline finishes.

Conditions for the post actions are:

- always
- success
- faulure

```
post {
    success {
        echo 'Success!'
    }
    failure {
        echo 'Failed!'
    }
}
```

## WHEN

Stage conditions - run stage on condition. It can be useful when running stage for certain branch only. It takes logical operators like AND / OR etc.

```
stage('Deploy') {
            when {
                branch 'main'
            }
```
## ENV

Jenkinsfile provides some variables and also allows defining custom variables used later in pipelines. List of vars provided by Jenkins is available under **http://jenkins-domain.tld/env-vars.html**.

To define your own var:

```
environment {
   YOUR_VAR = 'x'
}

stages {
   stage("build") {
      steps {
         echo 'building ${YOUR_VAR}' <- single quotes - interpolation, double quotes string
      }
   }
}
```

Credentials can also be passed as variables:
1. credentials('credentials-ID') <- scope of whole pipeline
2. withCredentials() <- scope of block

## TOOLS

Tools attribute in Jenkins is for declaring which tools need to be installed before running pipeline.  It’s a declarative-only feature that integrates with Global Tool Configuration in Jenkins.

```
tools {
        maven 'maven-3.9'
}
```
## PARAMETERS

Parameters let you customize a build at runtime. When a job has parameters, Jenkins shows a form before the build starts so users (or automation) can pass values in.

```
parameters {
        string(name: 'VERSION', defaultValue: '', description: 'version to deploy on prod)
        choice(name: 'VERSION, choices: ['1.1.01, '1.2.0', '1.3.0'] description: ''])
        booleanParam(name: 'executeTests', defaultValue: true, description '')
    }

stage("test") {
   when {
      params.executeTests
   }
}
```

Once Jenkins has parameters defined and fetched you can run build with "Build with Parameters".

## EXTERNAL SCRIPTS

Instead of putting all logic in a single Jenkinsfile, you can move reusable logic into .groovy files and load them into the pipeline. Envs and parameters are also available for groovy scripts.

```
def gv

pipeline {
   agent any

   stages {
      stage("init") {
         steps {
            script {
               gv = load "script.groovy"
            }
         }
      }
   }
}
```

## USER INPUT PARAMETERS

User input can be done in two ways:

1. As input block:
```
            input {
                message "Select the environment to deploy to"
                ok "Done"
                parameters{
                    choice(name: 'ONE', choices: ['dev', 'staging', 'prod'], description: '')
                    choice(name: 'TWO', choices: ['dev', 'staging', 'prod'], description: '')
                }
            }
            steps {
                script {
                    gv.deployApp()
                    echo "Deploying to ${ONE}"
                    echo "Deploying to ${TWO}"
                }
            }
```

2. As environment variable:

```
script {
                    env.ENV = input message: "Select the environment to deploy to", ok: "Done", parameters: [choice(name: 'ONE', choices: ['dev', 'staging', 'prod'], description: '')]
                    gv.deployApp()
                    echo "Deploying to ${ENV}"
}

```

## INTRO TO MULTIBRANCH PIPELINE

In software development process when using multiple git branches along with Jenkins for CI/CD you may need to run test,build and deploy master branch and test all the others without deploying - Jenkins allow multibranch pipelines do accomplish that.

Jenkinsfile is usually shared between all the branches in Multibranch Pipelines.

Variable of BRANCH_NAME is specific to multibranch pipelines.

When working with multiple branches you have them matched by regular expression and then Jenkins automatically creates pipeline for each branch.

Dealing with multiple staged build does not require starting all the stages from the beginning - Jenkins allows starting from specific Stage.

## CREDENTIALS IN JENKINS

There are multiple kinds of credentials that can be used with Jenkins: username and password, SSH username with private key, certificate etc. Plugins can bring new types of credentials in Jenkins.

In Jenkins credentials have scopes:

1. System - available on Jenkins server but not for Jenkins jobs. Only internal operations of Jenkins
2. Global - available everywhere
3. Limited to a project - available only with multibranch pipeline (comes from "Folder plugin")

## JENKINS SHARED LIBRARY

In Jenkins, a shared library is used to centralize, reuse, and standardize pipeline code across multiple projects. Instead of copy-pasting Groovy scripts into every Jenkinsfile, you define them once and reuse them everywhere. Clean, scalable, and much easier to maintain. In other words it extends pipeline - has its own repository, is written in groovy, reference is shared in Jenkinsfile.


Once you have such shared library it can be made available globally or for the project.

Structure of shared library in Jenkins:

- vars folder - functions that are called from Jenkinsfile, each function has its own Groovy file
- src - helper code
- resources - used for external libraries and non groovy files

Shared library doesn't necessarily need to be defined globally - it can be called directly in Jenkinsfile:

```
library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
   [$class: 'GitSCMSource',
   remote: 'https://xyz.com/repo.git',
   credentialsId: 'GitHub-token'])
```

##  WEBHOOKS

Jenkins allows to trigger jobs in multiple way:

1. Manual trigger
2. Automatic trigger
3. Scheduled trigger

Automatic triggers use webhooks. To configure webhook in Github you need to:

1. Add webhook in repository settings - main part is the "Payload URL" that goes like https://jenkins-addres.xyz:8080/github-webhook/
2. Select "GitHub hook trigger for GITScm polling" in pipeline configuration

For multibranch pipelines a plugin is needed - "Multibranch Scan Webhook Trigger". It is configured by selecting "Scan Multibranch Pipeline Triggers -> Scan by webhook".

## VERSIONING THE APPLICATION

Software versioning is the practice of assigning structured identifiers (version numbers or names) to software releases so developers and users can track changes, improvements, bug fixes, and compatibility over time.

Usually version is structured into three parts: major.minor.patch

**MAJOR** - big, possibly braking changes and not backward-compatibile

**MINOR** - new but backward-compatibile changes, API features

**PATCH** - minor changes and bug fixes, doesn't change API

Sometimes suffixes such as SNAPSHOT, RC, RELEASE etc. are uses.

In CI/CD process it is common to automatically increase version inside build automation.

Bumping versions for specific type of the app:

JAVA MAVEN:

There's a plugin - build helper that operates on pom.xml that would for example bump up patch value in version and copy pom.xml to a new version of a file:

```
$ mvn build-helper:parse-version versions:set
\-DnewVersion=\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion}
```

To bump up the minor version:

```
$ mvn build-helper:parse-version versions:set
\-DnewVersion=\${parsedVersion.majorVersion}.\${parsedVersion.nextMinorVersion}.\${parsedVersion.IncrementalVersion}
```

If you add:

`versions:commit` in the end old pom.xml file will be removed

Whole process can be done automatically in Jenkins pipeline - both version bump and later committing version change to repository.

When you have webhooks it can result in endless loop of building the pipeline. Jenkins plugins comes to play - "Ignore Committer Strategy" or for Github - "GitHub Commit Skip SCM Behaviour". The first plugin is configured under "Build Strategies" in pipeline configuration. There you can define what e-mail addresses of the committers would not trigger the pipeline.
