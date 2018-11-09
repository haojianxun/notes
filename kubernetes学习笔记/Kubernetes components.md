# Kubernetes components

The processes that govern Kubernetes can be split into 3 groups:

- Master components
- Node components
- Addons

## Master components

Master components provide the cluster’s control plane.

Master components make global decisions about the cluster (for example, scheduling), and detecting and responding to cluster events (starting up a new pod when a replication controller’s ‘replicas’ field is unsatisfied).

Master components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all master components on the same machine, and do not run user containers on this machine.

The components belonging to the Master components group are:

### kube-apiserver

Component on the master that exposes the Kubernetes API. It is the front-end for the Kubernetes control plane.

### etcd

Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data.

Always have a backup plan for etcd’s data for your Kubernetes cluster.

### kube-scheduler

Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.

Factors taken into account for scheduling decisions include individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference and deadlines.

### kube-controller-manager

Component on the master that runs controllers.

A controller is defined as: A *control loop* that watches the shared state of the cluster through the api server and makes changes attempting to move the current state towards the desired state.

Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

- Node Controller: Responsible for noticing and responding when nodes go down.
- Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system
- Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
- Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.

### cloud-controller-manager

cloud-controller-manager runs controllers that interact with the underlying cloud providers.

The cloud-controller-manager binary is an alpha feature introduced in Kubernetes release 1.6.

cloud-controller-manager aims to provide an abstraction layer for various cloud providers, allowing you to easily run K8s on the cloud provider of your choice.

## Node components

Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

A node is an actual (virtual or real) machine providing resources on which K8s schedules and runs your application.

### kubelet

An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.

The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy.

The kubelet doesn’t manage containers which were not created by Kubernetes.

### kube-proxy

kube-proxy enables the Kubernetes service abstraction by maintaining network rules on the host and performing connection forwarding.

### Container Runtime

The container runtime is the software that is responsible for running containers. Kubernetes supports several runtimes: Docker, rkt, runc and any OCI runtime-spec implementation.

## Addons

Addons are pods and services that implement various other non-essential cluster features.

Examples would be a DNS service, a Web dashboard, resource monitors, logging frameworks etc

An extended list of available addons is available [here](https://kubernetes.io/docs/concepts/cluster-administration/addons/).