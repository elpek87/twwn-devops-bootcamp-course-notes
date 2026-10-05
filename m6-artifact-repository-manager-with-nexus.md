![Logo](assets/devops-bootcamp-logo.png)

## ARTIFACT REPOSITORY MANAGER WITH NEXUS

**Artifact Repository** is a centralized system for storing, versioning, and distributing build outputs (called artifacts) across your delivery pipeline.

**Artifact Repository Manager** is a tool that stores, organizes, secures, and governs access to build artifacts (multiple types) and integrates with CI/CD pipelines to publish, promote, and retrieve them. There are public repository managers such as Maven Central Repository or self-hosted ones such as Nexus.

Nexus makes a good fit for Repository Manager because:

1. Multi-format support
2. Flexible API
3. Easy integration with LDAP
4. Cleanup policies

# Installation and setup

1. Nexus is java based app so it needs JRE to run it.

`# apt install java-17-jre-headless`

2. Download and untar nexus archive:

`# wget https://download.sonatype.com/nexus/3/nexus-3.88.0-08-linux-x86_64.tar.gz`

`$ tar xf nexus-3.88.0-08-linux-x86_64.tar.gz`

It creates two directories - nexus-version (application directory) and sonatype-work (data directory and customizations such as plugins etc.).

3. Create user for nexus and chown folders for nexus

`# useradd -s /bin/bash nexus`
`# chown -R nexus:nexus /opt/nexus-3.88.0-08 /opt/sonatype-work`

4. Set run_as_user property to make nexus run as nexus user. In newer versions you do so by editing bin/nexus

`# user to execute as; optional but recommended to set`
`run_as_user='nexus'`

5. Start nexus:

`$ /opt/nexus-3.88.0-08/bin/nexus start`

# Nexus usage

Nexus has three main repository types:

- proxy - used to mirror remote repository
- hosted - used to store artifact published by you
- group - aggregates repos under single endpoint

# Publishing artifacts to Nexus

For JAVA applications to publish an app to remote package repository you need to:

1. Enable remote repository plugins and specify remote repository in build.gradle (for Gradle) and pom.xml (for Maven)
2. If remote repository uses user authentication then it has to be defined in gradle.properties (Grdadle) and ~/.m2/settings.xml (Maven)

To publish the artifact:

`$ gradle publish`

`$ mvn deploy`

# Nexus API

The Nexus API is most commonly used to programmatically interact with Sonatype Nexus Repository Manager, instead of clicking around the web UI.

In practice, it lets tools, scripts, and CI/CD pipelines manage artifacts, repositories, users, and security automatically.

To interact with the API you use curl:

`$ curl -u user:password -X GET 'http://<IP>:8081/service/rest/v1/<endpoint>'`

Most common API calls:

1. Show components of specific repository:
 `$ curl -u user:password -X GET 'http://<IP>:8081/service/rest/v1/components?repository=<repository>'`

2. Show details of specific component:

`$ curl -u user:password -X GET 'http://<IP>:8081/service/rest/v1/components/<component-ID>`

# Nexus Blob Store

In Nexus Repository, a Blob Store is basically where the actual files (artifacts) are physically stored. It can be either file store, Google Cloud Storage or S3.

You can have multiple blob stores - what is the benefit? Control, safety, and sanity.

# Nexus - Component vs Asset

In Nexus **component** is a logical unit you think of as "the artifact" and the **asset** is a physical file stored in the repository. In Docker each layer is separate asset.

# Nexus Cleanup Policies and Scheduled Tasks

It is strongly advised to configure Cleanup Policy and assign it to repositories to keep its size in check. By default files that are affected by cleanup policy are not deleted - they're just marked for deletion (soft deletion). To reclaim space you need to add a task to "Compact blob store".
