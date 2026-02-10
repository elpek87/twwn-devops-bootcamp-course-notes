# CONTAINER ORCHESTRATION WITH KUBERNETES

## INTRO TO KUBERNETES

Kubernetes (often shortened to K8s) is a container orchestration platform.

What features do orchestration tools offer?

- high availability or no downtime
- scalability or high performance
- disaster recovery (backup and restore)

## KUBERNETES COMPONENTS

**POD** - like a wrapper around containers that lets them run together:
- smallest unit in K8S,
- abstraction over container
- usually one application per pod
- each pod gets its own IP address
- pods are ephemeral
- new IP on restart

**SVC** (Service) like a virtual IP and load balancer in front of the pods:
- static IP address
- lifecycle of service and pods are separate

**ING** (Ingress) - like a reverse proxy at the edge of a cluster (done by nginx, traefik etc.):
- manages external http/https access to services
- routes traffic based on hostnames / paths
- handles tls/https
- centralizes routing rules
- requires an ingress controller

**CM** (ConfigMap) - external .env file or config managed by k8s:
- stores app settings, feature flags, command-line arguments and config files (nginx.conf, app.yaml etc.)
- is read by applications at runtime in form of env vars or files mounted into pods
- stores non-sensitive configuration data

**SECRET** - secure vault entry injected into pods
- stores sensitive data such as passwords, ssh keys, api tokens etc.:
- access can be restricted using RBAC
- can be encrypted with (not by default - usually by 3rd party tools)
- can be encrypted at rest (when sitting on disk)

**VOL** (Volumes) - how k8s gives containers shared or persistent storage:
- since pods are ephemeral thanks to volumes we get a way to have data persistence
- k8s does not manage sata persistence like data replication, backups etc.

**DEPLOYMENT** - manages stateless pods
- abstraction of pods - all of the CRUD happens on this level and everything else follows
- blueprint for "applications" in pods
- this is what replicates and gets created not the pods themselves

**REPLICASET** - maintains the desired number of pod replicas at all times

**STATEFULSET** - manages stateful pods that need stable identity and storage
- meant for databases but more complex
- DBs are often hosted outside k8s

**DEAMONSET** - ensures that a copy of a Pod runs on every node (or a selected set of nodes):
- calculates how meny replicas are needed based on existing nodes
- it scales up and down with nodes

## KUBERNETES ARCHITECTURE

In Kubernetes, a cluster is split into Control Plane and Nodes (Workers). This separation is what lets Kubernetes decide what should run vs actually run it.

**NODE** (WORKER NODE) - a machine (VM or physical) that runs your application Pods:
- each node has multiple pods on it
- worker nodes do the actual work in cluster


Each node must have 3 processes running on them:
    - **container runtime** (docker, containerd, cri-o etc.)
    - **kubelet** - ensures that pods assigned to node run and are fine. Interacts with container and node
    - **kube proxy** - handles network routing to Services

**CONTROL PLANE** - it's the brain of a cluster, decides and plans but do not run

Each node must have 4 processes running on them:
    - **api server** cluster gateway (entrypoint) that gets and validates initial requests and gatekeeper (ensures requesters are authenticated)
    - **scheduler** decides which node should run each. Talks to kubelet to run pods.
    - **control manager** - detects cluster state changes. Talks to scheduler.
    - **etcd** - key value store. Cluster brain - all cluster changes are stored in this database - not app data.

Example setup:

Small, entry cluster would usually have 2 x Control Plane Nodes and 3 x Worker Nodes.  Control Plane needs less resources than Worker Nodes.

Adding new Control Plane and Worker Node servers is quite easy:
1. Get a new machine
2. Install all the control plane / worker node processes
3. Join the cluster.

## MINIKUBE AND KUBECTL - LOCAL KUBERNETES CLUSTER

Minikube is a tool that lets you run Kubernetes locally on your own machine. Both Control Plane and Worker Node run on same machine.

The *kubectl* is the command-line tool you use to talk to a Kubernetes cluster. Interacts with cluster API.

## Basic kubectl commands

Basic kubectl command syntax is: `kubectl <verb> <resource> <name> [flags]`

Getting information on elements of cluster is dome by "get" verb eg.:

`$ kubectl get pods`

Creating component of a cluster is done by "create" verb:

`$ kubectl create deployment nginx-deployment --image nginx`

Editing components of a cluster is done by "edit" verb:

`$ kubectl edit deployment nginx-deployment`

Debugging is done by "logs" verb:

`$ kubectl logs nginx-deployment-54668589f5-czpqv`

Describing of an element of a cluster is done by "describe" verb:

`$ kubectl describe pod mongo-deployment-5dc7f4b7d7-mxt5b`

Executing command using container shell is done by exec verb:

`$ kubectl exec -it mongo-deployment-5dc7f4b7d7-mxt5b -- /bin/bash`

Deleting deployment:

`$ kubectl delete deployments nginx-deployment`

Configuration files for k8s:

`$ kubectl apply -f config-file.yaml`

## YAML CONFIGURATION FILE

Every configuration files in k8s has three parts:

1. Metadata of a component
2. Specification of a component (specific to component)
3. Status (auto generated by k8s)

Kubernetes configuration files are YAML based. Sometimes you may need a validator. Configuration files should be kept with a code (usually git).

Pods are configured in specification/template section:

```
  template:
    metadata:
      labels:
        app: my-app
```

Labels = key–value tags attached to Kubernetes objects. These like like tags on AWS resources or labels on Docker containers. Almost everything can be labeled: pods, deployments, services etc.

```
labels:
  app: my-app
  env: prod
  tier: backend
```

Selectors = queries that match labels and matches objects that have specific labels.

```
selector:
  matchLabels:
    app: my-app
```

Deployments are connected to Services using selectors that match labels.

Another important section of configuration files are ports:

1. Service has port that is accessible
2. Service knows how to connect to port of the container that is defined as targetPort.

Working with deployments and services configuration file is usually done using:

`$ kubectl apply -f file.yaml`

More information on pods can be obtained with:

`$ kubectl get pods -o wide`

To see the status section (generated by k8s):

`kubectl get deployment nginx-deployment -o yaml `

Apart from the status parts some of metadata is also updated (timestaps, ids). If such deployment config dump would be used somewhere else it needs to be cleaned up first!

To delete launched deployment and service simply use:

`$ kubectl delete -f nginx-deployment.yml`

`$ kubectl delete -f nginx-service.yml`

## COMPLETE DEMO PROJECT - DEPLOYING APPLICATION IN KUBERNETES CLUSTER

Roadmap for the demo part:

1. Create mongo-db deployment and *internal* service (no external requests)
2. Create mongo-express deployment and service to connect to mongo-db service
3. Create ConfigMap that contains DB URL
4. Create Secret that contains credentials to the database
5. Create external service to mongo-express deployment

Passwords should not be stored in config files pushed to the repository! In secrets file use base64 encoded values.

Order of creating configuration files matter - especially if one depends on the other.

In external service we need to expose the port:
```
type: LoadBanalcer # just a name
```
and
```
nodePort: xxx # port number from range of 30000-32767
```

In minikube it works a bit different and to access the external service you need to run:
`$  minikube service mongo-express-service`

## NAMESPACES - ORGANIZING COMPONENTS

Namespaces (NS) in Kubernetes are basically virtual clusters inside one physical cluster. They help you organize, isolate, and control resources so things don’t turn into a YAML jungle.

By default k8s provides 4 namespaces:

1. kube-system - system processes, control plane and kubectl processes
2. kube-public - publicly available data, ConfigMap that contains cluster information
3. kube-node-lease - heartbeats of nodes, determine availability of nodes
4. default - that's where everything goes unless you create a new namespace

To create a namespace:
`$ kubectl create namespace name-of-namespace`

You can also use namespace configuration file:

```
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

Why namespaces?

1. To have resources logically grouped in Namespaces
2. Many teams and same name of the deployment would cause data overwriting (conflict avoidance)
3. Resource sharing: Staging and Development
4. Resource sharing: Blue/Green Deployment
5. Access and resource limits on Namespaces

What to keep in mind?

1. In most cases you can't access most resources from another Namespace - each Namespace must define separate ConfigMaps / Secrets
2. What can be accessed are services - in CMs you call them through service-name.namespace
3. There are some elements that are global and can't be isolated by Namespaces - volumes, nodes

```
$ kubectl api-resources --namespaced=false
```
```
$ kubectl api-resources --namespaced=true
```
To create component in a namespace for given let's say CM:

1. In metadata section (preferred):
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
  namespace: my-namespace
data:
  database_url: mongodb-service.database
```
Or with:

`$ kubectl apply -f mongo-configmap.yaml --namespace=my-namespace`

To get info on some component in a namespace use:

`$ kubectl get configmap -n my-namespace`

To change default active namespace:

`$ kubectl config set-context --current --namespace=my-namespace`


Another way is to use `kubectx` to switch between contexts (clusters) and `kubectx` to switch between namespaces.

## SERVICES - CONNECTING APPLICATIONS INSIDE CLUSTER

A Service in Kubernetes is a stable network endpoint that lets you reliably access a group of Pods — even though Pods themselves are ephemeral and their IPs change all the time.


What does a service do?

1. Selects Pods using labels
2. Gives them a stable virtual IP (ClusterIP)
3. Load-balances traffic across healthy Pods
4. Automatically updates when Pods change


Types of Services in k8s:

1. **ClusterIP Services**

A ClusterIP Service is the default and most common type of Kubernetes Service. It gives your app a stable, internal IP address that is reachable only inside the cluster. Services don’t “own” Pods — they select them.

Service:
```
selector:
  app: microservice-one

+ taegetPort: xxx
```

Pod:
```
labels:
  app: microservice-one
```

Use case: frontend → backend, app → database

When service gets created k8s creates endpoint object with the same name of the service that keeps track of which pods are the members/endpoints of the service.

2. **Multi-Port Services**

Multi-Port Services are simply Services (including ClusterIP) that expose more than one port for the same set of Pods.

Use case: App traffic + metrics

In this type of services ports are ***named***.

3. **Headless Service**

A Headless Service is a Kubernetes Service without a virtual IP. Instead of load-balancing traffic, it gives you direct DNS records for each Pod.

```
spec:
  clusterIP: None
```
Use case: databases

- DNS resolves to multiple Pod IPs
- Client chooses which Pod to talk to
- No load balancing by Kubernetes

How does client figure out IPs of each pod?

1. API call to k8s API Server - inefficient, ties app to API in an unhealthy way
2. DNS Lookup - it will return pods IP addresses.

There can be multiple services to same group of pods - ClusterIP and Headless (depends on the use case).

4. **NodePort Service**

A NodePort Service exposes your application on a static port on every node in the cluster, so it can be reached from outside the cluster using a node’s IP address. By default, Services (ClusterIP) are internal-only. NodePort is the simplest way to make a Service reachable externally without cloud load balancers or Ingress.

Use case: local development, learning or bare metal clusters with bo cloud provider

5. **LoadBalancer Service**

A LoadBalancer Service exposes your application to the outside world using a real external load balancer, usually provided automatically by your cloud provider. Extension of NodePort.

Use case: foundation for ingress

## INGRESS - CONNECTING TO APPLICATIONS OUTSIDE CLUSTER

External Service = “Expose this app on a network port”

Ingress = “Route HTTP/HTTPS traffic smartly to many apps using one entry point”

Exposing apps using LoadBalancer Service per app can get messy and expensive. Using NodePorts it's even worse - lacks TLS, nice URLs etc.

Ingres comes to the rescue and provides:

- single entry point
- clean URLs
- TLS termination
- host and path based routing

For ingress to work you need an implementation of ingress - Ingress Controller which by default is an NGINX Ingress Controller but there are more of them like Traefik, HAProxy etc.

In ingress there is also "Default backend" which is the fallback service that receives traffic when no Ingress rule matches an incoming request.

Example use case: If none of the ingress rules match then send the request to an error-service.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  defaultBackend:
    service:
      name: error-service
      port:
        number: 80
```

Ingress can be configured with multiple paths for same host (like domain.com/xyz) or multiple hosts - domains, subdomains - like xyz.domain.com.

To configure TLS in ingress you use tls key in yaml and define secret (of type "tls") name that holds your SSL certificate and key. Cert and key need to base64 encoded. The secret component needs to be in the same namespace as the Ingress.

You can even force HTTPS - for nginx it goes like this:
```
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

## VOLUMES - PERSISTING APPLICATION DATA

Since containers are ephemeral once such container restarts data is gone. In terms of an app that stores uploaded files, writes logs or uses database such data loss is not acceptable.

Sometimes what is needed is:

- storage that doesn't depend on the pod lifecycle
- storage that must be available on all nodes
- storage that needs to survive even if the cluster crashes

A **PersistentVolume** is a real piece of storage in the cluster. It's created via YAML file.

Storage in k8s is not managed by the cluster - it only uses what administrators provide as an external plugin to the cluster. It can be cloud storage backend, NFS or some other network FS or mix of them.

It's created via YAML file. Depending on storage type spec attributes can differ.

Persistent Volumes are not namespaced - they are available to all of rhe elements of the cluster.

Local storage is usually tied to specific node and may not survive cluster crash - it's recommended to use remote storage in production systems that needs data persistence.

**PersistentVolumeClaim (PVC)**  is a request for storage. Also created with YAML file.

Admin provisions storage then user creates claim to PV.

In pod configuration you reference PVC in volumes section (claimName). PVCs must exist in same namespace as the pods that use them. Once desired volume is found its get mounted into pod and then into container.

**ConfigMap** and **Secret** are also volume types. Both are local volumes that are managed by k8s. You just create CM or Secret component and then mount that into your pod / container.

**StorageClass** defines how storage should be created. It's used to provision persistent volumes dynamically.

Storage classes are defined by ***provisioner*** keyword. It's also requested by PVC.

## CONFIGMAP AND SECRET VOLUME TYPES

You use a ConfigMap volume when your application expects configuration as files, not just environment variables. Contents can be key value, multi value, full config format and even some binary - very flexible.

Then there's Secret volume type which stores sensitive configuration data.

## STATEFULSET - DEPLOYING STATEFUL APPLICATIONS

A StatefulSet in Kubernetes is a workload type designed for stateful applications — apps that need stable identity and stable storage over time.

Examples of such applications are databases or distributed systems (kafka).

Stateless applications in k8s are deployed using Deployment and stateful applications are deployed using StatefulSet component.

How does Deployment and StatefulSet differ in this case?

Deployment -> "Run N copies of this container. I don’t care which is which.”

StatefulSet -> "I want N specific instances, each with its own identity (predictable name, dns) and disk (not shared storage). **Sticky identity**."

For StatefulSet you use persistent volumes on remote storage.

In StatefulSet deletion takes place in reveresed order - in Deployment pods get deleted randomly.

In StatefulSet cloning and data sync is not automatically done by k8s.

Stateful apps are not the best candidate to run them in containers.

## MANAGED KUBERNETES SERVICE

Unmanaged (Self-Managed) Kubernetes is when you install and operate Kubernetes yourself.

Managed Kubernetes is when the cloud provider runs Kubernetes for you.

Some well known managed k8s are: LKE, WKS, GKE, AKS

In such managed environments you typically care only for worker nodes - everything else is on provider's side.

Cloud providers also provide LoadBalancers for managed k8s.

Migrations between providers may not always be easy - usage of services specific to given provider - vendor lock-in.

## HELM - PACKAGE MANAGER FOR KUBERNETES

Helm is the package manager for Kubernetes. Instead of manually applying multiple YAML files (Deployment, Service, Ingress, ConfigMap, Secret) Helm lets you bundle them together (Helm Charts), parameterize them, version them, install/upgrade/rollback them with one command.

You can search for helm charts using:
`$ helm search <keyword>`

Or browse Artifact Hub or some other public repositories.

Helm uses Go templates to generate Kubernetes YAML dynamically - there's no need to write multiple separate files for services that are almost identical.

What is important here is: `values.yaml` that poulate templating macros such as `{{ Values.container.name }}` etc.

This is very practical for CI/CD or deploying same applications on different clusters like prod, dev etc.

Helm Chart Structure:
- Chart.yaml (meta info about chart)
- values.yaml (values for template file)
- charts/ (chart dependencies)
- templates/ (template files)
- ...

To deploy Helm Chart into k8s:
`$ helm install <chartname>`

Values are injected into template into multiple ways:

1. `$ helm install --values=my-values.yaml <chartname>` <- this can override values.yaml (defaults)
2. `$ helm install --set version=2.0.0 <chartname>` <- directly from the command line is also possible

Release Management = Helm tracking, versioning, and controlling deployments over time.

Helm doesn’t just apply YAML and walk away. It remembers what it deployed, how, and when.

Thanks to this you can use: `$ helm upgrade <chartname>` and if something goes wrong: `$ helm rollback <chartname>`. Revision numbers are always incremented by 1.
