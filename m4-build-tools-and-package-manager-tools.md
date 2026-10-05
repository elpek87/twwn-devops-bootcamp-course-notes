![Logo](assets/devops-bootcamp-logo.png)

## BUILD TOOLS AND PACKAGE MANAGER TOOLS

Building code - process of transforming source code into something runnable. It consumes dependencies and produces artifacts - these are used to deploy.

*Example*: Apache Maven, Gradle, Make

Package management - element responsible for getting and versioning dependencies needed to build end product.

*Example*: npm, pip

Artifact repository - storage for the end product of build process (and dependencies).

*Example*: JFrog, Nexus

Build is done using tools specific to used programming language. During the process they fetch and install the dependencies then compile and compress your code.

# Building the code and running the app (java)

Gradle - to build code in gradle you'd use:

`$ gradle build`

For gradle dependencies are defined in *gradle.build* file

Maven - to build code in maven you'd use:

`$ mvn install`

For maven dependencies are defined in *pom.xml* file

To run java application you'd use:
`$ java -jar <name_of_jar_file>`

# Building the code and running the app (JavaScript)

For JavaScript the tools that you'd use to install needed dependencies is npm or yarn - both manage their dependencies using package.json. If project has separate frontend and backend they can either use separate package.json files or have everything defined in the single file. To install needed dependencies for the JS project use:

`$ npm install`

By default produced application archive (npm pack) DOES NOT include the dependencies - these need to be installed before running the app.

To run JS application you'd use:

`$ npm start`

# Artifacts vs Docker Images

Depending on language used you usually deal with different files when it comes to artifacts - JARs, WARs, ZIPs etc.

Thanks to Docker dealing with artifacts became easier - all you care about here is Docker Image - universal artifact. It can be created to prepare desired environment for the app and then run it in there exposing the app endpoint to the outside.
