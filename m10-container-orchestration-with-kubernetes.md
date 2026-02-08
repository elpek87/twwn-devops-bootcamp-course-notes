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
- abstraction of pods
- blueprint for "applications" in pods
- this is what replicates and gets created not the pods themselves

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
