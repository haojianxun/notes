# [kubernetes基本入门教程4]YAML FILE STRUCTURE

To create an object in Kubernetes, you must provide an object spec that describes its desired state

As mentioned before, objects in Kubernetes are “records of intent”

Thich means you provide a declarative description of your desired state and Kubernetes will work on getting your system in that state and maintaining it there.

In the example below we see an example of such an object.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

All Kubernetes objects have 4 mandatory fields, `apiVersion`, `kind`, `metadata` and `spec`

the `apiVersion` field specifies version of the Kubernetes API you’re using to create this object

Kubernetes uses several layers of apis to push new objects from concepts/testing (alpha) to tested, but not completely standardised (beta) to stable versions

Most of the objects in this intro course will fall into the stable `v1` version, but if you’re unsure you can allways lookup at the official [reference documentation](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.11/)

The second mandatory field is the `kind` field which specifies the type of object you want to create (field is case-sensitive)

In this case we’re creating a `Pod` object, but it could be any other kubernetes object, like `Service`, `ReplicationController`, `Deployment`, `ConfigMap`etc

The `metadata` field contains fields that uniquely identify an object, such as it’s name or the namespace it belongs to.

Another important fields in the `metadata` field is the `labels` map in which we store the labels we want applied

They allow you to loose coupling of your objects into organisational structure. You can use arbitrary labels, like `release`, `tier` `environment` etc

The final mandatory field is the `spec` field. It is unique to each `kind` of object created and specifies the details of our “intent

For instance, the minimum configuration required for the `Pod` type is the containers it will run.

As an example of all the fields available (and to get you used to referencing the docs) we recommend you having a look at the official [pod reference docs](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.11/#pod-v1-core)

As you can see, there are plenty of fields we can specify, but for now let’s look at our `containers` field.

The official `PodSpec` for the `containers` field says: List of containers belonging to the pod. Containers cannot currently be added or removed. There must be at least one container in a Pod. Cannot be updated.

In our example we see a list of one [Container](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.11/#container-v1-core)

The container we’re running is named `nginix`, is running an `nginx:1.7.9`image (in this case pulled from the official docker registry).

And has a `ContainerPort` defined, which is the specification of the port which is to be exposed to the outside.

We hope you’re comfortable editing and writing YAML files, if not, a quick reference can be found [here](https://learnxinyminutes.com/docs/yaml/)

Let’s see how comfortable are you with the topics covered in this lesson!

# TASK

fill out the `pod.yaml` file in your editor to create a pod named `redis`running one container with the image `redis`.
Run `kubectl apply -f pod.yaml` (after you’ve finished and SAVED!) to create the Pod



Great job! The examples will get more complicated later, but the principle is the same

`apiVersion` version of the API

 `kind` - the type of object 

`metadata` - name and other identifiers 

`spec` - specification for that object type

Another thing we’d like to point out is the ability to put several objects inside the same yaml file

Using the `---` separator will make `kubectl` treat the file like several files

You can use this mechanism to package things you often create together, like a Pod and a Service exposing it

