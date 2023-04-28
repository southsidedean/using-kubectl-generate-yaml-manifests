# Using `kubectl` to Generate YAML Manifests
**Tom Dean - 4/28/2023**

## Introduction

I'm currently studying to take the **CKAD** and **CKA** exams, and there remain a few key areas where I need to build my Kubernetes muscles. Coming from a Linux background (of many years), the cluster building and troubleshooting doesn't faze me. Creating declarative manifests, well, that's a different story.

Using the `kubectl` command imperatively to create Kubernetes manifests for declarative deployment, an important skill for both exams, lands at the top of that list. On the **CKAD** and **CKA** exams, *every second counts*, and this skill facilitates the *rapid and consistent generation* of **YAML** manifests that can *quickly* be edited and deployed to Kubernetes.

**What makes this skill so important?**

- Fast: *Seconds count.* Less reliant on finding an example in the docs, then copying it.
- Accurate: *No typos. No formatting errors.* Manifest works the first time.
- Easy: Let `kubectl` do the heavy lifting.

[Adrien Trouillaud](https://www.linkedin.com/in/trouillaud/) wrote an excellent Medium article, [Imperative vs. Declarative — a Kubernetes Tutorial](https://medium.com/payscale-tech/imperative-vs-declarative-a-kubernetes-tutorial-4be66c5d8914) which I discovered while researching this topic. Adrien's article, a comprehensive tutorial on how to leverage the imperative approach to creating objects in Kubernetes, clarified the topic for me. The article dates back to early 2019, so some of the syntax has changed, but the concepts in the article still ring true. I learned even more by digging into the commands and updating the syntax, where needed. *I would highly recommend giving it a read.*

In this tutorial, we're going to get *hands-on* with using the `kubectl` command *imperatively* to create some common objects as *declarative* YAML manifests.

***Let's dig in!***

## References

[Medium: Imperative vs. Declarative — a Kubernetes Tutorial](https://medium.com/payscale-tech/imperative-vs-declarative-a-kubernetes-tutorial-4be66c5d8914)

[Kubernetes: Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)

[Kubernetes: kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

[Kubernetes: Install Tools](https://kubernetes.io/docs/tasks/tools/)

[GitHub: cka-d-cluster-builder-lab](https://github.com/southsidedean/cka-d-cluster-builder-lab)

[GitHub: Deploying a Kubernetes In Docker (KIND) Cluster Using Podman on Ubuntu Linux](https://github.com/southsidedean/deploy-kind-using-podman-ubuntu)

[Kubernetes: Managing Kubernetes Objects Using Imperative Commands](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/)

[]()

[]()

[]()

[]()

## Prerequisites

[Kubernetes: Install Tools](https://kubernetes.io/docs/tasks/tools/)

[GitHub: cka-d-cluster-builder-lab](https://github.com/southsidedean/cka-d-cluster-builder-lab)

[GitHub: Deploying a Kubernetes In Docker (KIND) Cluster Using Podman on Ubuntu Linux](https://github.com/southsidedean/deploy-kind-using-podman-ubuntu)

*The examples in this guide come from an Ubuntu Linux 22.04 operating system environment with a two-node Kubernetes 1.26.1 cluster configured with `kubeadm`, using the [`cka-d-cluster-builder-lab`](https://github.com/southsidedean/cka-d-cluster-builder-lab) automation.*

You're going to need a Kubernetes environment to perform the steps in this tutorial.  If you have access to one already, you can use it.  If not, you'll need to create one.  I've provided some resources in the links in this section.

The first link, from the Kubernetes documentation, contains resources detailing several options for cluster installation.

I've also provided links to two of my GitHub repositories with tutorials, one for creating a `kubeadm` cluster using Terraform and KVM ([GitHub: cka-d-cluster-builder-lab](https://github.com/southsidedean/cka-d-cluster-builder-lab)) and a second for creating a Kubernetes In Docker, or KIND, cluster ([GitHub: Deploying a Kubernetes In Docker (KIND) Cluster Using Podman on Ubuntu Linux](https://github.com/southsidedean/deploy-kind-using-podman-ubuntu)).  Either of these will work for this tutorial.

When you have your cluster, check it out. Start with the high-level cluster information.

High-level information:
```bash
kubectl cluster-info
```

**Sample Output:**
```bash
$ kubectl cluster-info

Kubernetes control plane is running at https://10.0.1.132:6443
CoreDNS is running at https://10.0.1.132:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

You can use the `kubectl get` command to take a look at objects in your cluster.

Take a look at your Kubernetes nodes.

List of nodes, with detail:
```bash
kubectl get nodes -o wide
```

**Sample Output:**
```bash
$ kubectl get nodes -o wide

NAME               STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
control-plane-01   Ready    control-plane   6d22h   v1.26.1   10.0.1.132    <none>        Ubuntu 22.04.2 LTS   5.15.0-69-generic   containerd://1.6.20
worker-node-01     Ready    <none>          6d22h   v1.26.1   10.0.1.204    <none>        Ubuntu 22.04.2 LTS   5.15.0-69-generic   containerd://1.6.20
```

Take a look at *all the objects* on your cluster.

List all objects, in all namespaces:
```bash
kubectl get all -A
```

**Sample Output:**
```bash
$ kubectl get all -A

NAMESPACE     NAME                                           READY   STATUS    RESTARTS   AGE
kube-system   pod/calico-kube-controllers-57b57c56f-4hvfl    1/1     Running   0          6d22h
kube-system   pod/calico-node-s28hb                          1/1     Running   0          6d22h
kube-system   pod/calico-node-tqb6g                          1/1     Running   0          6d22h
kube-system   pod/coredns-787d4945fb-cg5wh                   1/1     Running   0          6d22h
kube-system   pod/coredns-787d4945fb-xdbll                   1/1     Running   0          6d22h
kube-system   pod/etcd-control-plane-01                      1/1     Running   0          6d22h
kube-system   pod/kube-apiserver-control-plane-01            1/1     Running   0          6d22h
kube-system   pod/kube-controller-manager-control-plane-01   1/1     Running   0          6d22h
kube-system   pod/kube-proxy-blgcf                           1/1     Running   0          6d22h
kube-system   pod/kube-proxy-t9wb4                           1/1     Running   0          6d22h
kube-system   pod/kube-scheduler-control-plane-01            1/1     Running   0          6d22h

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  6d22h
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   6d22h

NAMESPACE     NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/calico-node   2         2         2       2            2           kubernetes.io/os=linux   6d22h
kube-system   daemonset.apps/kube-proxy    2         2         2       2            2           kubernetes.io/os=linux   6d22h

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           6d22h
kube-system   deployment.apps/coredns                   2/2     2            2           6d22h

NAMESPACE     NAME                                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/calico-kube-controllers-57b57c56f   1         1         1       6d22h
kube-system   replicaset.apps/coredns-787d4945fb                  2         2         2       6d22h
```

List all namespaces:
```bash
kubectl get ns
```

**Sample Output:**
```bash
$ kubectl get ns
NAME              STATUS   AGE
default           Active   6d22h
kube-node-lease   Active   6d22h
kube-public       Active   6d22h
kube-system       Active   6d22h
```

***With a working cluster at your disposal, you're ready to proceed.***

## Exploring the `kubectl` Command

[Kubernetes: Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)

[Kubernetes: kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

The `kubectl` command, the primary tool you're going to use to interact with your Kubernetes cluster, both in this tutorial and the **CKA/CKAD** exams, provides a user-friendly, *or user-friendlier*, method of interacting with the Kubernetes API.

### Help in `kubectl`

Let's take a look at the `kubectl` command, starting at the top.

Top-level help:
```bash
kubectl --help
```

**Sample Output:**
```bash
$ kubectl --help

kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/

Basic Commands (Beginner):
  create          Create a resource from a file or from stdin
  expose          Take a replication controller, service, deployment or pod and expose it as a new Kubernetes service
  run             Run a particular image on the cluster
  set             Set specific features on objects

Basic Commands (Intermediate):
  explain         Get documentation for a resource
  get             Display one or many resources
  edit            Edit a resource on the server
  delete          Delete resources by file names, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout         Manage the rollout of a resource
  scale           Set a new size for a deployment, replica set, or replication controller
  autoscale       Auto-scale a deployment, replica set, stateful set, or replication controller

Cluster Management Commands:
  certificate     Modify certificate resources.
  cluster-info    Display cluster information
  top             Display resource (CPU/memory) usage
  cordon          Mark node as unschedulable
  uncordon        Mark node as schedulable
  drain           Drain node in preparation for maintenance
  taint           Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe        Show details of a specific resource or group of resources
  logs            Print the logs for a container in a pod
  attach          Attach to a running container
  exec            Execute a command in a container
  port-forward    Forward one or more local ports to a pod
  proxy           Run a proxy to the Kubernetes API server
  cp              Copy files and directories to and from containers
  auth            Inspect authorization
  debug           Create debugging sessions for troubleshooting workloads and nodes
  events          List events

Advanced Commands:
  diff            Diff the live version against a would-be applied version
  apply           Apply a configuration to a resource by file name or stdin
  patch           Update fields of a resource
  replace         Replace a resource by file name or stdin
  wait            Experimental: Wait for a specific condition on one or many resources
  kustomize       Build a kustomization target from a directory or URL.

Settings Commands:
  label           Update the labels on a resource
  annotate        Update the annotations on a resource
  completion      Output shell completion code for the specified shell (bash, zsh, fish, or powershell)

Other Commands:
  alpha           Commands for features in alpha
  api-resources   Print the supported API resources on the server
  api-versions    Print the supported API versions on the server, in the form of "group/version"
  config          Modify kubeconfig files
  plugin          Provides utilities for interacting with plugins
  version         Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

The `kubectl --help` command provides you with a great high-level view of all the commands available to `kubectl`. You can dig down further by adding the command.

Drilling down, into the `kubectl run` command:
```bash
kubectl run --help
```

**Sample Output:**
```bash
$ kubectl run --help

Create and run a particular image in a pod.

Examples:
  # Start a nginx pod
  kubectl run nginx --image=nginx
  
  # Start a hazelcast pod and let the container expose port 5701
  kubectl run hazelcast --image=hazelcast/hazelcast --port=5701
  
  # Start a hazelcast pod and set environment variables "DNS_DOMAIN=cluster" and "POD_NAMESPACE=default" in the
container
  kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"
  
  # Start a hazelcast pod and set labels "app=hazelcast" and "env=prod" in the container
  kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=hazelcast,env=prod"
  
  # Dry run; print the corresponding API objects without creating them
  kubectl run nginx --image=nginx --dry-run=client
  
  # Start a nginx pod, but overload the spec with a partial set of values parsed from JSON
  kubectl run nginx --image=nginx --overrides='{ "apiVersion": "v1", "spec": { ... } }'
  
  # Start a busybox pod and keep it in the foreground, don't restart it if it exits
  kubectl run -i -t busybox --image=busybox --restart=Never
  
  # Start the nginx pod using the default command, but use custom arguments (arg1 .. argN) for that command
  kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>
  
  # Start the nginx pod using a different command and custom arguments
  kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>

Options:
    --allow-missing-template-keys=true:
	If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to
	golang and jsonpath output formats.

    --annotations=[]:
	Annotations to apply to the pod.

    --attach=false:
	If true, wait for the Pod to start running, and then attach to the Pod as if 'kubectl attach ...' were called.
	Default false, unless '-i/--stdin' is set, in which case the default is true. With '--restart=Never' the exit
	code of the container process is returned.

    --command=false:
	If true and extra arguments are present, use them as the 'command' field in the container, rather than the
	'args' field which is the default.

    --dry-run='none':
	Must be "none", "server", or "client". If client strategy, only print the object that would be sent, without
	sending it. If server strategy, submit server-side request without persisting the resource.

    --env=[]:
	Environment variables to set in the container.

    --expose=false:
	If true, create a ClusterIP service associated with the pod.  Requires `--port`.

    --field-manager='kubectl-run':
	Name of the manager used to track field ownership.

    --image='':
	The image for the container to run.

    --image-pull-policy='':
	The image pull policy for the container.  If left empty, this value will not be specified by the client and
	defaulted by the server.

    -l, --labels='':
	Comma separated labels to apply to the pod. Will override previous values.

    --leave-stdin-open=false:
	If the pod is started in interactive mode or with stdin, leave stdin open after the first attach completes. By
	default, stdin will be closed after the first attach completes.

    -o, --output='':
	Output format. One of: (json, yaml, name, go-template, go-template-file, template, templatefile, jsonpath,
	jsonpath-as-json, jsonpath-file).

    --override-type='merge':
	The method used to override the generated object: json, merge, or strategic.

    --overrides='':
	An inline JSON override for the generated object. If this is non-empty, it is used to override the generated
	object. Requires that the object supply a valid apiVersion field.

    --pod-running-timeout=1m0s:
	The length of time (like 5s, 2m, or 3h, higher than zero) to wait until at least one pod is running

    --port='':
	The port that this container exposes.

    --privileged=false:
	If true, run the container in privileged mode.

    -q, --quiet=false:
	If true, suppress prompt messages.

    --restart='Always':
	The restart policy for this Pod.  Legal values [Always, OnFailure, Never].

    --rm=false:
	If true, delete the pod after it exits.  Only valid when attaching to the container, e.g. with '--attach' or
	with '-i/--stdin'.

    --save-config=false:
	If true, the configuration of current object will be saved in its annotation. Otherwise, the annotation will
	be unchanged. This flag is useful when you want to perform kubectl apply on this object in the future.

    --show-managed-fields=false:
	If true, keep the managedFields when printing objects in JSON or YAML format.

    -i, --stdin=false:
	Keep stdin open on the container in the pod, even if nothing is attached.

    --template='':
	Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format
	is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

    -t, --tty=false:
	Allocate a TTY for the container in the pod.

Usage:
  kubectl run NAME --image=image [--env="key=value"] [--port=port] [--dry-run=server|client] [--overrides=inline-json]
[--command] -- [COMMAND] [args...] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

This view gives you a high-level explanation of the command, usage examples and the syntax and switches for the command.

Go ahead, play around, get familiar with the help functionality in the `kubectl` command. It's the fastest documentation for `kubectl`, both when taking the exams and for day-to-day use.

### Kubernetes API Resources

Kubernetes API resources, another part of the `kubectl` command you should understand, can be viewed using the `kubectl api-resources` command.

Listing all available API resources:
```bash
kubectl api-resources
```

**Sample Output:**
```bash
$ kubectl api-resources

NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
persistentvolumeclaims            pvc          v1                                     true         PersistentVolumeClaim
persistentvolumes                 pv           v1                                     false        PersistentVolume
pods                              po           v1                                     true         Pod
podtemplates                                   v1                                     true         PodTemplate
replicationcontrollers            rc           v1                                     true         ReplicationController
resourcequotas                    quota        v1                                     true         ResourceQuota
secrets                                        v1                                     true         Secret
serviceaccounts                   sa           v1                                     true         ServiceAccount
services                          svc          v1                                     true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io/v1              false        APIService
controllerrevisions                            apps/v1                                true         ControllerRevision
daemonsets                        ds           apps/v1                                true         DaemonSet
deployments                       deploy       apps/v1                                true         Deployment
replicasets                       rs           apps/v1                                true         ReplicaSet
statefulsets                      sts          apps/v1                                true         StatefulSet
tokenreviews                                   authentication.k8s.io/v1               false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io/v1                true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io/v1                false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io/v1                false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io/v1                false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling/v2                         true         HorizontalPodAutoscaler
cronjobs                          cj           batch/v1                               true         CronJob
jobs                                           batch/v1                               true         Job
certificatesigningrequests        csr          certificates.k8s.io/v1                 false        CertificateSigningRequest
leases                                         coordination.k8s.io/v1                 true         Lease
bgpconfigurations                              crd.projectcalico.org/v1               false        BGPConfiguration
bgppeers                                       crd.projectcalico.org/v1               false        BGPPeer
blockaffinities                                crd.projectcalico.org/v1               false        BlockAffinity
caliconodestatuses                             crd.projectcalico.org/v1               false        CalicoNodeStatus
clusterinformations                            crd.projectcalico.org/v1               false        ClusterInformation
felixconfigurations                            crd.projectcalico.org/v1               false        FelixConfiguration
globalnetworkpolicies                          crd.projectcalico.org/v1               false        GlobalNetworkPolicy
globalnetworksets                              crd.projectcalico.org/v1               false        GlobalNetworkSet
hostendpoints                                  crd.projectcalico.org/v1               false        HostEndpoint
ipamblocks                                     crd.projectcalico.org/v1               false        IPAMBlock
ipamconfigs                                    crd.projectcalico.org/v1               false        IPAMConfig
ipamhandles                                    crd.projectcalico.org/v1               false        IPAMHandle
ippools                                        crd.projectcalico.org/v1               false        IPPool
ipreservations                                 crd.projectcalico.org/v1               false        IPReservation
kubecontrollersconfigurations                  crd.projectcalico.org/v1               false        KubeControllersConfiguration
networkpolicies                                crd.projectcalico.org/v1               true         NetworkPolicy
networksets                                    crd.projectcalico.org/v1               true         NetworkSet
endpointslices                                 discovery.k8s.io/v1                    true         EndpointSlice
events                            ev           events.k8s.io/v1                       true         Event
flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta3   false        FlowSchema
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta3   false        PriorityLevelConfiguration
ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
ingresses                         ing          networking.k8s.io/v1                   true         Ingress
networkpolicies                   netpol       networking.k8s.io/v1                   true         NetworkPolicy
runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass
poddisruptionbudgets              pdb          policy/v1                              true         PodDisruptionBudget
clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io/v1           true         RoleBinding
roles                                          rbac.authorization.k8s.io/v1           true         Role
priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
csinodes                                       storage.k8s.io/v1                      false        CSINode
csistoragecapacities                           storage.k8s.io/v1                      true         CSIStorageCapacity
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment
```

*Note that many API resource types have shortnames that you can use to speed up typing commands.*

You can dig deeper into these API resources using the `kubectl explain` command, which serves as a great reference when creating manifests.

More information about `deployments` API resource, via the `kubectl explain` command:
```bash
kubectl explain deploy
```

**Sample Output:**
```bash
$ kubectl explain deploy

KIND:     Deployment
VERSION:  apps/v1

DESCRIPTION:
     Deployment enables declarative updates for Pods and ReplicaSets.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>
     Specification of the desired behavior of the Deployment.

   status	<Object>
     Most recently observed status of the Deployment.
```

Use `kubectl get deploy` to list all `deployments`:
```bash
kubectl get deploy
```

**Sample Output:**
```bash
$ kubectl get deploy

No resources found in default namespace.
```

You're not seeing any `deployments` because, without providing a namespace, you're only looking at the `default` namespace.

List all `deployments` in *all* namespaces:
```bash
kubectl get deploy -A
```

**Sample Output:**
```bash
$ kubectl get deploy -A

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   calico-kube-controllers   1/1     1            1           6d23h
kube-system   coredns                   2/2     2            2           6d23h
```

You can see that the previous example shows two `deployments` in the the `kube-system` namespace.

Sometimes you want to see more than one API resource. You can do that, by separating the resources with commas, as shown in the following example.

List all `deployments`, `replicasets` and `pods`, in all namespaces:
```bash
kubectl get deploy,rs,pod -A
```

**Sample Output:**
```bash
$ kubectl get deploy,rs,pod -A

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           6d23h
kube-system   deployment.apps/coredns                   2/2     2            2           6d23h

NAMESPACE     NAME                                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/calico-kube-controllers-57b57c56f   1         1         1       6d23h
kube-system   replicaset.apps/coredns-787d4945fb                  2         2         2       6d23h

NAMESPACE     NAME                                           READY   STATUS    RESTARTS   AGE
kube-system   pod/calico-kube-controllers-57b57c56f-4hvfl    1/1     Running   0          6d23h
kube-system   pod/calico-node-s28hb                          1/1     Running   0          6d23h
kube-system   pod/calico-node-tqb6g                          1/1     Running   0          6d23h
kube-system   pod/coredns-787d4945fb-cg5wh                   1/1     Running   0          6d23h
kube-system   pod/coredns-787d4945fb-xdbll                   1/1     Running   0          6d23h
kube-system   pod/etcd-control-plane-01                      1/1     Running   0          6d23h
kube-system   pod/kube-apiserver-control-plane-01            1/1     Running   0          6d23h
kube-system   pod/kube-controller-manager-control-plane-01   1/1     Running   0          6d23h
kube-system   pod/kube-proxy-blgcf                           1/1     Running   0          6d23h
kube-system   pod/kube-proxy-t9wb4                           1/1     Running   0          6d23h
kube-system   pod/kube-scheduler-control-plane-01            1/1     Running   0          6d23h
```

The previous example shows how you can neatly check on multiple API resource types in a single command.

Take some time, explore the `kubectl` command, the built-in help documentation, API resources and how to use the `kubectl explain` command as a resource.

**AGAIN:**

[Adrien Trouillaud](https://www.linkedin.com/in/trouillaud/) wrote an excellent Medium article, [Imperative vs. Declarative — a Kubernetes Tutorial](https://medium.com/payscale-tech/imperative-vs-declarative-a-kubernetes-tutorial-4be66c5d8914) which I discovered while researching this topic. Adrien's article, a comprehensive tutorial on how to leverage the imperative approach to creating objects in Kubernetes, clarified the topic for me. The article dates back to early 2019, so some of the syntax has changed, but the concepts in the article still ring true. I learned even more by digging into the commands and updating the syntax, where needed. *I would highly recommend giving it a read.*

***Now that you've warmed up with some `kubectl` exercises, it's time to put `kubectl` to use.***

## Creating a Pod Imperatively Using the `kubectl run` Command

[Kubernetes: Managing Kubernetes Objects Using Imperative Commands](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/)

Quite often, you just want to create a *straightforward* Kubernetes object, like a Pod, without having to write a manifest and feed it to Kubernetes. You can, using the `kubectl` command imperatively.

Say you've been presented with the following request:

***"Create a *single* pod, named `nginx-pod`, using the image `nginx:latest`, in the `imperative` namespace, with `port 80` exposed."***

Create the `nginx-pod` imperatively:
```bash
kubectl run nginx-pod --image nginx:latest --namespace imperative --port=80
```

You will get the message `Error from server (NotFound): namespaces "imperative" not found`. You need to create the namespace. This can also be done imperatively using `kubectl create`.

Create the `imperative` namespace imperatively:
```bash
kubectl create namespace imperative
```

You should see `namespace/imperative created`.

Try creating your Pod again.  Create the `nginx-pod` imperatively:
```bash
kubectl run nginx-pod --image nginx:latest --namespace imperative --port=80
```

You should get `pod/nginx-pod created`.

Checking our work:
```bash
kubectl get pod --namespace imperative -o wide
```

**Sample Output:**
```bash
$ kubectl get pod --namespace imperative -o wide

NAME        READY   STATUS    RESTARTS   AGE   IP                NODE             NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          10s   192.168.126.201   worker-node-01   <none>           <none>
```

You can dig deeper and get more information using the `kubectl describe` command.

More information on our pod:
```bash
kubectl describe pod nginx-pod --namespace imperative
```

**Sample Output:**
```bash
$ kubectl describe pod nginx-pod --namespace imperative

Name:             nginx-pod
Namespace:        imperative
Priority:         0
Service Account:  default
Node:             worker-node-01/10.0.1.204
Start Time:       Thu, 27 Apr 2023 18:49:16 +0000
Labels:           run=nginx-pod
Annotations:      cni.projectcalico.org/containerID: 69fa1b2e11018aa307d1ee64fd99c3ba6c1d91f0f74ffa0a40eb93c0560ae041
                  cni.projectcalico.org/podIP: 192.168.126.201/32
                  cni.projectcalico.org/podIPs: 192.168.126.201/32
Status:           Running
IP:               192.168.126.201
IPs:
  IP:  192.168.126.201
Containers:
  nginx-pod:
    Container ID:   containerd://e66a03ed5344e3c6927369774a35df446ee93c937ee038453afd5e297e79c5d7
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:63b44e8ddb83d5dd8020327c1f40436e37a6fffd3ef2498a6204df23be6e7e94
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 27 Apr 2023 18:49:18 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vbfgp (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-vbfgp:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  51s   default-scheduler  Successfully assigned imperative/nginx-pod to worker-node-01
  Normal  Pulling    50s   kubelet            Pulling image "nginx:latest"
  Normal  Pulled     50s   kubelet            Successfully pulled image "nginx:latest" in 399.613484ms (399.621888ms including waiting)
  Normal  Created    50s   kubelet            Created container nginx-pod
  Normal  Started    49s   kubelet            Started container nginx-pod
```

Using `kubectl describe` provides you with a thorough description of the Pod.

Kubernetes says the status of our `nginx-pod` pod is good. Let's look at it from another angle. Use the `curl` command to test the NGINX server in your pod.

Test using `curl`, replace with your pod's IP address:
```bash
curl http://<POD_IP>
```

**Sample Output:**
```bash
$ curl http://192.168.126.201

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

The NGINX web server in our `nginx-pod` Pod responds with the default web page. Everything appears to be working as expected.

What if you want to use `kubectl` imperatively, but want to generate a declarative manifest and not immediately create the objects in Kubernetes?

***Let's take a look at how you can use the `kubectl` command imperatively to create manifests!***

## Generating a Namespace Manifest Using the `kubectl create` Command

You created your `imperative` namespace to hold the objects you want to create *imperatively*, using `kubectl`. Now, you're going to start creating objects *declarively*, so you're going to put those into the `declarative` namespace.

Creating a namespace provides a great example of how you can leverage the imperative to create the declarative. You can use the imperative command `kubectl create namespace declarative` with the `--dry-run=client` and `--output=yaml` swiches, and redirect the output into a file.

Create the `declarative` namespace:
```bash
kubectl create namespace declarative --dry-run=client --output=yaml > declarative-ns.yaml
```

Checking our work:
```bash
cat declarative-ns.yaml
```

**Sample Output:**
```bash
$ cat declarative-ns.yaml

apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: declarative
spec: {}
status: {}
```

Now, let's use the YAML manifest to create our namespace:
```bash
kubectl create -f declarative-ns.yaml
```

**Sample Output:**
```bash
$ kubectl create -f declarative-ns.yaml

namespace/declarative created
```

Checking our work:
```bash
kubectl get ns
```

**Sample Output:**
```bash
$ kubectl get ns

NAME              STATUS   AGE
declarative       Active   23s
default           Active   7d3h
imperative        Active   5m7s
kube-node-lease   Active   7d3h
kube-public       Active   7d3h
kube-system       Active   7d3h
```



***Transition***

## Generating a Pod Manifest Using the `kubectl run` Command



```bash
kubectl run nginx-pod --image nginx:latest --namespace declarative --port=80 --dry-run=client --output=yaml > nginx-pod.yaml
```

Checking our work:
```bash
cat nginx-pod.yaml
```

**Sample Output:**
```bash
$ cat nginx-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-pod
  name: nginx-pod
  namespace: declarative
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f nginx-pod.yaml
```

You should get `pod/nginx-pod created`.

Checking our work:
```bash
kubectl get pod --namespace declarative -o wide
```

**Sample Output:**
```bash
$ kubectl get pod --namespace declarative -o wide

NAME        READY   STATUS    RESTARTS   AGE   IP                NODE             NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          13s   192.168.126.202   worker-node-01   <none>           <none>
```

Kubernetes reports the status of our `nginx-pod` pod as `Running`. Let's look at it from another angle. Use the `curl` command to test the NGINX server in your pod.

Test using `curl`, replace with your pod's IP address:
```bash
curl http://<POD_IP>
```

**Sample Output:**
```bash
$ curl http://192.168.126.202

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

The NGINX web server in our `nginx-pod` Pod responds with the default web page. Everything appears to be working as expected.



***Transition***

## Creating Deployment Manifests Using the `kubectl create` Command




```bash
kubectl create deploy --help
```

**Sample Output:**
```bash
$ kubectl create deploy --help

Create a deployment with the specified name.

Aliases:
deployment, deploy

Examples:
  # Create a deployment named my-dep that runs the busybox image
  kubectl create deployment my-dep --image=busybox
  
  # Create a deployment with a command
  kubectl create deployment my-dep --image=busybox -- date
  
  # Create a deployment named my-dep that runs the nginx image with 3 replicas
  kubectl create deployment my-dep --image=nginx --replicas=3
  
  # Create a deployment named my-dep that runs the busybox image and expose port
5701
  kubectl create deployment my-dep --image=busybox --port=5701

Options:
    --allow-missing-template-keys=true:
	If true, ignore any errors in templates when a field or map key is
	missing in the template. Only applies to golang and jsonpath output
	formats.

    --dry-run='none':
	Must be "none", "server", or "client". If client strategy, only print
	the object that would be sent, without sending it. If server strategy,
	submit server-side request without persisting the resource.

    --field-manager='kubectl-create':
	Name of the manager used to track field ownership.

    --image=[]:
	Image names to run.

    -o, --output='':
	Output format. One of: (json, yaml, name, go-template,
	go-template-file, template, templatefile, jsonpath, jsonpath-as-json,
	jsonpath-file).

    --port=-1:
	The port that this container exposes.

    -r, --replicas=1:
	Number of replicas to create. Default is 1.

    --save-config=false:
	If true, the configuration of current object will be saved in its
	annotation. Otherwise, the annotation will be unchanged. This flag is
	useful when you want to perform kubectl apply on this object in the
	future.

    --show-managed-fields=false:
	If true, keep the managedFields when printing objects in JSON or YAML
	format.

    --template='':
	Template string or path to template file to use when -o=go-template,
	-o=go-template-file. The template format is golang templates
	[http://golang.org/pkg/text/template/#pkg-overview].

    --validate='strict':
	Must be one of: strict (or true), warn, ignore (or false). 		"true" or
	"strict" will use a schema to validate the input and fail the request
	if invalid. It will perform server side validation if
	ServerSideFieldValidation is enabled on the api-server, but will fall
	back to less reliable client-side validation if not. 		"warn" will
	warn about unknown or duplicate fields without blocking the request if
	server-side field validation is enabled on the API server, and behave
	as "ignore" otherwise. 		"false" or "ignore" will not perform any
	schema validation, silently dropping any unknown or duplicate fields.

Usage:
  kubectl create deployment NAME --image=image -- [COMMAND] [args...] [options]

Use "kubectl options" for a list of global command-line options (applies to all
commands).
```


Create the `my-nginx-deployment` namespace:
```bash
kubectl create namespace my-nginx-namespace --dry-run=client --output=yaml > my-nginx-namespace.yaml
```

Checking our work:
```bash
cat my-nginx-namespace.yaml
```

**Sample Output:**
```bash
$ cat my-nginx-namespace.yaml

apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: my-nginx-namespace
spec: {}
status: {}
```

```bash
kubectl create deployment my-nginx-deployment --image=nginx:latest --namespace my-nginx-namespace --replicas=3 --port=80 --dry-run=client --output=yaml > my-nginx-deployment.yaml
```

```bash
cat my-nginx-deployment.yaml
```

**Sample Output:**
```bash
$ cat my-nginx-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: my-nginx-deployment
  name: my-nginx-deployment
  namespace: my-nginx-namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: my-nginx-deployment
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
status: {}
```

```bash
kubectl create -f my-nginx-namespace.yaml ; kubectl create -f my-nginx-deployment.yaml
```

**Sample Output:**
```bash
$ kubectl create -f my-nginx-namespace.yaml ; kubectl create -f my-nginx-deployment.yaml

namespace/my-nginx-namespace created
deployment.apps/my-nginx-deployment created
```

```bash
kubectl get all -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl get all -n my-nginx-namespace

NAME                                      READY   STATUS    RESTARTS   AGE
pod/my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          55s
pod/my-nginx-deployment-9cbcd46b4-7nmn8   1/1     Running   0          55s
pod/my-nginx-deployment-9cbcd46b4-xt89j   1/1     Running   0          55s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx-deployment   3/3     3            3           55s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-deployment-9cbcd46b4   3         3         3       55s
```

```bash
kubectl get pods -o wide -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl get pods -o wide -n my-nginx-namespace

NAME                                  READY   STATUS    RESTARTS   AGE     IP                NODE               NOMINATED NODE   READINESS GATES
my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          2m10s   192.168.173.196   control-plane-01   <none>           <none>
my-nginx-deployment-9cbcd46b4-7nmn8   1/1     Running   0          2m10s   192.168.126.204   worker-node-01     <none>           <none>
my-nginx-deployment-9cbcd46b4-xt89j   1/1     Running   0          2m10s   192.168.126.203   worker-node-01     <none>           <none>
```

Test using `curl`, replace with any of your three pods; IP addresses:
```bash
curl http://<POD_IP>
```

**Sample Output:**
```bash
$ curl http://192.168.126.203

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

*Repeat as desired, with the other IP addresses.*

The NGINX web server in our `nginx-pod` Pods respond with the default web page. Everything appears to be working as expected with our `my-nginx-deployment` deployment.



***Transition***

## Creating a Service Manifest Using the `kubectl create` Command



```bash
kubectl create svc --help
```

**Sample Output:**
```bash
$ kubectl create svc --help

Create a service using a specified subcommand.

Aliases:
service, svc

Available Commands:
  clusterip      Create a ClusterIP service
  externalname   Create an ExternalName service
  loadbalancer   Create a LoadBalancer service
  nodeport       Create a NodePort service

Usage:
  kubectl create service [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all
commands).
```



```bash
kubectl create svc clusterip --help
```

**Sample Output:**
```bash

```

**Sample Output:**
```bash
$ kubectl get svc -A

NAMESPACE     NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  7d4h
kube-system   kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   7d4h
```

```bash
kubectl create svc --help
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```



***Transition***

## Scaling a Replica Set Using the `kubectl scale` Command



```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```



***Transition***

## Adding Labels Using the `kubectl label` Command



```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```



***Transition***

## Checking Logs Using `kubectl`



```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```


***Transition***

## Exporting Running Kubernetes Objects to a Manifest Using `kubectl`



```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```


***Transition***

## Deleting Kubernetes Objects Using `kubectl`



```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```



***Transition***

## Section



```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```

```bash
kubectl 
```

**Sample Output:**
```bash

```



***Transition***

## Practice Exercises



### Exercise 1

***Create a *single* pod, imperatively, named ``, using the image ``, in the `` namespace, with `port ` exposed.***

[Exercise 1: Solution](solutions/solution-exercise-1.md)

### Exercise 2



[Exercise 2: Solution](solutions/solution-exercise-2.md)


### Exercise 3



[Exercise 3: Solution](solutions/solution-exercise-3.md)


### Exercise 4



[Exercise 4: Solution](solutions/solution-exercise-4.md)


### Exercise 5



[Exercise 5: Solution](solutions/solution-exercise-5.md)



***Transition***

## Summary



Enjoy!

*Tom Dean*
