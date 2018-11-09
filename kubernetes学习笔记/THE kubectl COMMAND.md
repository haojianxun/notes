# THE kubectl COMMAND

As we saw in previous lessons, the `kubectl` will be your main tool for maintaining and inspecting your cluster

In this lesson we’ll cover the basic use-cases of the command and show you the principles behind its design

If you’re interested in specific subcommands you can always have a look at the provided [reference documentation](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

The general syntax of the `kubectl` command is: `kubectl [command] [TYPE] [NAME] [flags]`

The `[command]` specifies the operation you want to perform, for instance `get`, `describe`, `delete`, `create` etc

The `type` and `name` pairs specify the target objects to which to apply the operation.

For instance if you’d like to inspect the state of the pod with the name `logger`, you’d run `kubectl describe pod logger`

```
command         description
---------------------------------------------------------------------
annotate        Add or update the annotations of one or more resources.
api-versions    List the API versions that are available.
apply           Apply a configuration change to a resource from a file or stdin.
attach          Attach to a running container either to view the output stream or interact with the container (stdin).
autoscale       Automatically scale the set of pods that are managed by a replication controller.
cluster-info    Display endpoint information about the master and services in the cluster.
config          Modifies kubeconfig files. See the individual subcommands for details.
create          Create one or more resources from a file or stdin.
delete          Delete resources either from a file, stdin, or specifying label selectors, names, resource selectors, or resources.
describe        Display the detailed state of one or more resources.
edit            Edit and update the definition of one or more resources on the server by using the default editor.
exec            Execute a command against a container in a pod,
explain         Get documentation of various resources. For instance pods, nodes, services, etc.
expose          Expose a replication controller, service, or pod as a new Kubernetes service.
get             List one or more resources.
label           Add or update the labels of one or more resources.
logs            Print the logs for a container in a pod.
patch           Update one or more fields of a resource by using the strategic merge patch process.
port-forward    Forward one or more local ports to a pod.
proxy           Run a proxy to the Kubernetes API server.
replace         Replace a resource from a file or stdin.
rolling-update  Perform a rolling update by gradually replacing the specified replication controller and its pods.
run             Run a specified image on the cluster.
scale           Update the size of the specified replication controller.
stop            Deprecated; Instead, see kubectl delete.
version         Display the Kubernetes version running on the client and server.
```



The `type` field is case-insesitive and you can specify the type in singular, plural or abbreviated form

```
name                          | abbreviation
----------------------------------------------
apiservices	
certificatesigningrequests	    csr
clusters	
clusterrolebindings	
clusterroles	
componentstatuses	              cs
configmaps	                    cm
controllerrevisions	
cronjobs	
customresourcedefinition	      crd
daemonsets	                    ds
deployments	                    deploy
endpoints	                      ep
events	                        ev
horizontalpodautoscalers	      hpa
ingresses	                      ing
jobs	
limitranges	                    limits
namespaces	                    ns
networkpolicies	                netpol
nodes	                          no
persistentvolumeclaims	        pvc
persistentvolumes	              pv
poddisruptionbudget	            pdb
podpreset	
pods	                          po
podsecuritypolicies	            psp
podtemplates	
replicasets	                    rs
replicationcontrollers	        rc
resourcequotas	                quota
rolebindings	
roles	
secrets	
serviceaccounts	                sa
services	                      svc
statefulsets	
storageclasses	
```

For instance, the following commands accomplish the same thing: 

- `kubectl get pods logger`
-  `kubectl get pod logger` 
- `kubectl get po logger`

For a list of supported resource types, please have a look [here](https://kubernetes.io/docs/reference/kubectl/overview/#resource-types)

If you omit the `name` field, you target all objects of that type. `kubectl get services` will list all services

To target multiple individual objects of the same type you can use the `TYPE1 name1 name2 name<#>` form: `kubectl get pod logger deleteme`

To target multiple objects of the different types use the `TYPE1/name1 TYPE1/name2 TYPE<#>/name<#>` form: `kubectl get pod/logger service/nginx-service`

Let’s run through a few exercises to see how `kubectl` behaves. (We recommend keeping the [command list](https://kubernetes.io/docs/reference/kubectl/overview/#operations) in another tab open)

# TASK

To start off, in your console, use the `describe` command to get info about the pod named `logger`

# TASK

Using the `delete` command, delete the service named `deleteme`

# TASK

Using the `logs` command, get the logs from the pod named `logger`
(note: the command doesn’t use standard format, check reference docs)

```
kubectl logs POD [-c CONTAINER] [--follow] [flags]
```



The next one is a bit tricky and will require you to lookup the info from the [reference docs.](https://kubernetes.io/docs/reference/kubectl/overview/#operations)

```
name                          | abbreviation
----------------------------------------------
apiservices	
certificatesigningrequests	    csr
clusters	
clusterrolebindings	
clusterroles	
componentstatuses	              cs
configmaps	                    cm
controllerrevisions	
cronjobs	
customresourcedefinition	      crd
daemonsets	                    ds
deployments	                    deploy
endpoints	                      ep
events	                        ev
horizontalpodautoscalers	      hpa
ingresses	                      ing
jobs	
limitranges	                    limits
namespaces	                    ns
networkpolicies	                netpol
nodes	                          no
persistentvolumeclaims	        pvc
persistentvolumes	              pv
poddisruptionbudget	            pdb
podpreset	
pods	                          po
podsecuritypolicies	            psp
podtemplates	
replicasets	                    rs
replicationcontrollers	        rc
resourcequotas	                quota
rolebindings	
roles	
secrets	
serviceaccounts	                sa
services	                      svc
statefulsets	
storageclasses	
```

# TASK

Using the `scale` command target the `deployment` named `test` to decrease the number of replicas to `1`
(hint: `--replicas` flag

```
kubectl scale (-f FILENAME \| TYPE NAME \| TYPE/NAME) --replicas=COUNT [--resource-version=version] [--current-replicas=count] [flags]
```

Another thing we can use to target cluster objects are their labels using the `-l` flag

For instance, to target all services with the label `delete=no` you can run `kubectl get services -l delete=no`

# TASK

Delete all services with the label `delete=yes`

Great! We’ll go deeper into specific commands in later lessons, let’s just have a quick look into formatting the output of the `kubectl` command.

# FORMATTING OUTPUT

Output format of most commands can be specified using the `-o` flag `kubectl [command] [TYPE] [NAME] -o=<output_format>`

```
-o=custom-columns=<spec>          Print a table using a comma separated list of custom columns.
-o=custom-columns-file=<filename>	Print a table using the custom columns template in the <filename> file.
-o=json	                          Output a JSON formatted API object.
-o=jsonpath=<template>            Print the fields defined in a jsonpath expression.
-o=jsonpath-file=<filename>       Print the fields defined by the jsonpath expression in the <filename> file.
-o=name                           Print only the resource name and nothing else.
-o=wide	                          Output in the plain-text format with any additional information. For pods, the node name is included.
-o=yaml	                          Output a YAML formatted API object.
```

Supported output formats can be found [here](https://kubernetes.io/docs/reference/kubectl/overview/#output-options)

For instance, to get output in json you’d run `kubectl get po logger -o json`

One thing we’d like to point out is the jsonpath support. It’s a powerful concept, especially for scripting purposes

We’ll be covering it in more advanced lessons, but if you’re interested you can have a look at the [official docs](https://kubernetes.io/docs/reference/kubectl/jsonpath/)

For instance, it allows you to pick and format specific fields of your output, like: `kubectl get po -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.resourceVersion}{"\n"}'`

We hope you have a general overview of the `kubectl` command now, as we’ll be using it a lot soon