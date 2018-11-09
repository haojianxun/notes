# INTERACTING WITH KUBERNETES

To work with Kubernetes, you use Kubernetes API objects to describe your cluster’s desired state

By state we mean what applications or other workloads you want to run, what container images they use, the number of replicas, what network and disk resources you want to make available, and more

You set your desired state by creating objects using the Kubernetes API, typically via the command-line interface, kubectl, or directly using the K8s API

Once you’ve set your desired state, the Kubernetes Control Plane works to make the cluster’s current state match the desired state.

To do so, Kubernetes performs a variety of tasks automatically–such as starting or restarting containers, scaling the number of replicas of a given application, and more.

As we mentioned before, there are a series of control loops running (Controllers) running inside k8s monitoring current state and attempting to move the system into the desired state

# Kubernetes objects

Kubernetes contains a number of abstractions that represent the state of your system

These abstractions include deployed containerized applications and workloads, their associated network and disk resources, and other information about what your cluster is doing

These abstractions are represented by `objects` in the Kubernetes API

A Kubernetes object is a “record of intent”–once you create the object, the Kubernetes system will constantly work to ensure that object exists.

By creating an object, you’re effectively telling the Kubernetes system what you want your cluster’s workload to look like; this is your cluster’s desired state.

In the editor below, we can see such an object.

The object used in this example is of type Pod (a basic “work unit” of k8s). (We’ll explore pods in more detail later)

pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

For now it’s enough to know that a Pod encapsulates one or more containers, and would represent a service running inside your cluster.

In this case we’re running a pod with one container, an nginx container, which has a port open on port 80

Let’s try and send this object to Kubernetes and see what happens.

inspect `pod.yaml` on your filesystem and send it to the kubernetes API using `curl -X POST -H "Content-Type: application/yaml" --data-binary "@pod.yaml" localhost:9992/api/v1/namespaces/default/pods`

返回的情况是:

```
magic@magic:~/msb-task/task$ curl -X POST -H "Content-Type: application/yaml" --data-binary "@pod.yaml" localhost:9992/api/v1/namespaces/default/pods
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "nginx",
    "namespace": "default",
    "selfLink": "/api/v1/namespaces/default/pods/nginx",
    "uid": "99a02ac9-e420-11e8-b73f-b6cd5a9ad314",
    "resourceVersion": "60852",
    "creationTimestamp": "2018-11-09T13:08:50Z"
  },
  "spec": {
    "volumes": [
      {
        "name": "default-token-9dh9m",
        "secret": {
          "secretName": "default-token-9dh9m",
          "defaultMode": 420
        }
      }
    ],
    "containers": [
      {
        "name": "nginx",
        "image": "nginx:1.7.9",
        "ports": [
          {
            "containerPort": 80,
            "protocol": "TCP"
          }
        ],
        "resources": {

        },
        "volumeMounts": [
          {
            "name": "default-token-9dh9m",
            "readOnly": true,
            "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
          }
        ],
        "terminationMessagePath": "/dev/termination-log",
        "terminationMessagePolicy": "File",
        "imagePullPolicy": "IfNotPresent"
      }
    ],
    "restartPolicy": "Always",
    "terminationGracePeriodSeconds": 30,
    "dnsPolicy": "ClusterFirst",
    "serviceAccountName": "default",
    "serviceAccount": "default",
    "securityContext": {

    },
    "schedulerName": "default-scheduler",
    "tolerations": [
      {
        "key": "node.kubernetes.io/not-ready",
        "operator": "Exists",
        "effect": "NoExecute",
        "tolerationSeconds": 300
      },
      {
        "key": "node.kubernetes.io/unreachable",
        "operator": "Exists",
        "effect": "NoExecute",
        "tolerationSeconds": 300
      }
    ]
  },
  "status": {
    "phase": "Pending",
    "qosClass": "BestEffort"
  }
```

Perfect! Your pod is being created as per your specification.

At the top of your dashboard you can see the events being emitted by kubernetes (click them for more info)

As you might have assumed, k8s exposes a RESTful interface to its objects. Let’s see if other HTTP verbs work

Send a `DELETE` request to the created pod by running `curl -X DELETE http://localhost:9992/api/v1/namespaces/default/pods/nginx`

Great! We have deleted the running pod. While the API access is nice from a development perspective, there’s a better user-facing way to interact with k8s - the `kubectl` command.

The `kubectl` command communicates with the kube-api in your stead and is the standard way of interacting with the cluster through the cli

We’ll go deeper into specific sub-commands of `kubectl` later, but for now let’s see how you’d interact with the cluster using it

Let’s replicate the previous API-based scenario using kubectl

in your console run `kubectl apply -f pod.yaml` to create the pod specified in the yaml

Great! That was much simpler.

`kubectl` can also be used to inspect the current state of your cluster

To get information about the objects in your cluster, you’d use `kubectl get` or `kubectl describe` commands

To list all your running pods run `kubectl get pods`

To get information about a specific pod you can specify the name of the pod. Run: `kubectl get pods/nginx` to get basic info about the pod you just created

To get much more information use the `describe` verb. Run: `kubectl describe pods/nginx`

Kubectl also allows operations on the cluster itself. For instance run `kubectl cluster-info` To get information about current cluster

Let’s finish the exercise by deleting you pod. Run: `kubectl delete pods/nginx` To delete the pod from the cluster. In the background the same request is sent as when we sent it manually using `curl`

Great! We’ll use the `kubectl` command much more in the following exercises.

As always, the [kubernetes.io](http://kubernetes.io/) website provides a detailed [overview](https://kubernetes.io/docs/reference/kubectl/overview/) and [reference](https://kubernetes.io/docs/reference/kubectl/kubectl/) if you ever need a quick lookup

`kubectl` can do much more - create proxies to your API and services, execute commands inside your containers, view container logs and much more.

You’re slowly starting to interact with your k8s cluster, pretty soon you’ll be getting your hands dirty. Let’s continue to next lessons.