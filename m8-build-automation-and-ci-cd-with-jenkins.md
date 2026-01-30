# Build Automation & CI/CD with Jenkins

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
