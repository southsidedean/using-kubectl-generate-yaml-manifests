# Using `kubectl` to Generate YAML Manifests
**Tom Dean - 4/28/2023**

## Introduction

I'm currently studying to take the **CKAD** and **CKA** exams, and there remain a few key areas where I need to build my Kubernetes muscles. Coming from a Linux background (of many years), the cluster building and troubleshooting doesn't faze me. Creating declarative manifests, well, that's a different story.

Using the `kubectl` command imperatively to create Kubernetes manifests for declarative deployment, an important skill for both exams, lands at the top of that list. On the **CKAD** and **CKA** exams, *every second counts*, and this skill facilitates the *rapid and consistent generation* of **YAML** manifests that can *quickly* be edited and deployed to Kubernetes.

**What makes this skill so important?**

- Fast: *Seconds count.* Less reliant on finding an example in the docs, then copying it.
- Accurate: *No typos. No formatting errors.* Manifest works the first time.
- Easy: Let `kubectl` do the heavy lifting.

[Adrien Trouillaud](https://www.linkedin.com/in/trouillaud/) wrote an excellent Medium article, [Imperative vs. Declarative — a Kubernetes Tutorial](https://medium.com/payscale-tech/imperative-vs-declarative-a-kubernetes-tutorial-4be66c5d8914), which I discovered while researching this topic. Adrien's article, a comprehensive tutorial on how to leverage the imperative approach to creating objects in Kubernetes, clarified the topic for me. The article dates back to early 2019, so some of the syntax has changed, but the concepts in the article still ring true. I learned even more by digging into the commands and updating the syntax, where needed. *I would highly recommend giving it a read.*

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

[Kubernetes: Service](https://kubernetes.io/docs/concepts/services-networking/service/)

[Kubernetes: Use a Service to Access an Application in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/)

[Kubernetes: ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

[Kubernetes: Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

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

## Generating a Namespace Manifest Using the `kubectl create namespace` Command

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

## Creating Deployment Manifests Using the `kubectl create deploy` Command

[Kubernetes: Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

[Kubernetes: ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)



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

***Create a YAML manifest for a Namespace, named `my-nginx-namespace` in a file named `my-nginx-namespace.yaml`.***

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

***Create a YAML manifest for a Deployment, named `my-nginx-deployment`, using the `nginx-latest` container image, in the `my-nginx-namespace` namespace, with 3 replicas, exposing port 80, in a file named `my-nginx-deployment.yaml`.***

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
kubectl get pods -o wide -n my-nginx-namespace --show-labels
```

**Sample Output:**
```bash
$ kubectl get pods -o wide -n my-nginx-namespace --show-labels
NAME                                  READY   STATUS    RESTARTS   AGE   IP                NODE               NOMINATED NODE   READINESS GATES   LABELS
my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          20h   192.168.173.196   control-plane-01   <none>           <none>            app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-7nmn8   1/1     Running   0          20h   192.168.126.204   worker-node-01     <none>           <none>            app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-xt89j   1/1     Running   0          20h   192.168.126.203   worker-node-01     <none>           <none>            app=my-nginx-deployment,pod-template-hash=9cbcd46b4
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

## Creating a Service Manifest Using the `kubectl create service` and `kubectl set selector` Commands

[Kubernetes: Service](https://kubernetes.io/docs/concepts/services-networking/service/)

[Kubernetes: Use a Service to Access an Application in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/)

[Kubernetes: Managing Kubernetes Objects Using Imperative Commands](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/)




```bash
kubectl get service -A
```

**Sample Output:**
```bash
$ kubectl get service -A

NAMESPACE     NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  7d4h
kube-system   kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   7d4h
```

```bash
kubectl create service --help
```

**Sample Output:**
```bash
$ kubectl create service --help

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
kubectl create service clusterip --help
```

**Sample Output:**
```bash
$ kubectl create service clusterip --help

Create a ClusterIP service with the specified name.

Examples:
  # Create a new ClusterIP service named my-cs
  kubectl create service clusterip my-cs --tcp=5678:8080

  # Create a new ClusterIP service named my-cs (in headless mode)
  kubectl create service clusterip my-cs --clusterip="None"

Options:
    --allow-missing-template-keys=true:
	If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to
	golang and jsonpath output formats.

    --clusterip='':
	Assign your own ClusterIP or set to 'None' for a 'headless' service (no loadbalancing).

    --dry-run='none':
	Must be "none", "server", or "client". If client strategy, only print the object that would be sent, without
	sending it. If server strategy, submit server-side request without persisting the resource.

    --field-manager='kubectl-create':
	Name of the manager used to track field ownership.

    -o, --output='':
	Output format. One of: (json, yaml, name, go-template, go-template-file, template, templatefile, jsonpath,
	jsonpath-as-json, jsonpath-file).

    --save-config=false:
	If true, the configuration of current object will be saved in its annotation. Otherwise, the annotation will
	be unchanged. This flag is useful when you want to perform kubectl apply on this object in the future.

    --show-managed-fields=false:
	If true, keep the managedFields when printing objects in JSON or YAML format.

    --tcp=[]:
	Port pairs can be specified as '<port>:<targetPort>'.

    --template='':
	Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format
	is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

    --validate='strict':
	Must be one of: strict (or true), warn, ignore (or false). 		"true" or "strict" will use a schema to validate
	the input and fail the request if invalid. It will perform server side validation if ServerSideFieldValidation
	is enabled on the api-server, but will fall back to less reliable client-side validation if not. 		"warn" will
	warn about unknown or duplicate fields without blocking the request if server-side field validation is enabled
	on the API server, and behave as "ignore" otherwise. 		"false" or "ignore" will not perform any schema
	validation, silently dropping any unknown or duplicate fields.

Usage:
  kubectl create service clusterip NAME [--tcp=<port>:<targetPort>] [--dry-run=server|client|none] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```



```bash
kubectl set selector --help
```

**Sample Output:**
```bash
$ kubectl set selector --help

Set the selector on a resource. Note that the new selector will overwrite the old selector if the resource had one prior
to the invocation of 'set selector'.

 A selector must begin with a letter or number, and may contain letters, numbers, hyphens, dots, and underscores, up to
63 characters. If --resource-version is specified, then updates will use this resource version, otherwise the existing
resource-version will be used. Note: currently selectors can only be set on Service objects.

Examples:
  # Set the labels and selector before creating a deployment/service pair
  kubectl create service clusterip my-svc --clusterip="None" -o yaml --dry-run=client | kubectl set selector --local -f
- 'environment=qa' -o yaml | kubectl create -f -
  kubectl create deployment my-dep -o yaml --dry-run=client | kubectl label --local -f - environment=qa -o yaml |
kubectl create -f -

Options:
    --all=false:
	Select all resources in the namespace of the specified resource types

    --allow-missing-template-keys=true:
	If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to
	golang and jsonpath output formats.

    --dry-run='none':
	Must be "none", "server", or "client". If client strategy, only print the object that would be sent, without
	sending it. If server strategy, submit server-side request without persisting the resource.

    --field-manager='kubectl-set':
	Name of the manager used to track field ownership.

    -f, --filename=[]:
	identifying the resource.

    --local=false:
	If true, annotation will NOT contact api-server but run locally.

    -o, --output='':
	Output format. One of: (json, yaml, name, go-template, go-template-file, template, templatefile, jsonpath,
	jsonpath-as-json, jsonpath-file).

    -R, --recursive=true:
	Process the directory used in -f, --filename recursively. Useful when you want to manage related manifests
	organized within the same directory.

    --resource-version='':
	If non-empty, the selectors update will only succeed if this is the current resource-version for the object.
	Only valid when specifying a single resource.

    --show-managed-fields=false:
	If true, keep the managedFields when printing objects in JSON or YAML format.

    --template='':
	Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format
	is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

Usage:
  kubectl set selector (-f FILENAME | TYPE NAME) EXPRESSIONS [--resource-version=version] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```


```bash
kubectl create service clusterip my-nginx-service --tcp=8888:80 --namespace my-nginx-namespace --dry-run=client --output=yaml | kubectl set selector --local -f - 'app=my-nginx-deployment' --output=yaml > my-nginx-service.yaml
```

```bash
cat my-nginx-service.yaml
```

**Sample Output:**
```bash
$ cat my-nginx-service.yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: my-nginx-service
  name: my-nginx-service
  namespace: my-nginx-namespace
spec:
  ports:
  - name: 8888-80
    port: 8888
    protocol: TCP
    targetPort: 80
  selector:
    app: my-nginx-deployment
  type: ClusterIP
status:
  loadBalancer: {}
```

```bash
kubectl create -f my-nginx-service.yaml
```

**Sample Output:**
```bash
$ kubectl create -f my-nginx-service.yaml

service/my-nginx-service created
```


```bash
kubectl get service -A
```

**Sample Output:**
```bash
$ kubectl get service -A
NAMESPACE            NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default              kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP                  8d
kube-system          kube-dns           ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   8d
my-nginx-namespace   my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP                 6s
```


```bash
kubectl get all -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl get all -n my-nginx-namespace

NAME                                      READY   STATUS    RESTARTS   AGE
pod/my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          20h
pod/my-nginx-deployment-9cbcd46b4-7nmn8   1/1     Running   0          20h
pod/my-nginx-deployment-9cbcd46b4-xt89j   1/1     Running   0          20h

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP   3m35s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx-deployment   3/3     3            3           20h

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-deployment-9cbcd46b4   3         3         3       20h
```

```bash
curl http://10.111.212.138:8888
```

**Sample Output:**
```bash
$ curl http://10.111.212.138:8888

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



***Transition***

## Scaling a Deployment/ReplicaSet Using the `kubectl scale` Command

[Kubernetes: ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

[Kubernetes: Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)



```bash
kubectl get rs -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl get rs -n my-nginx-namespace

NAME                            DESIRED   CURRENT   READY   AGE
my-nginx-deployment-9cbcd46b4   3         3         3       21h
```

```bash
kubectl describe rs <YOUR_REPLICA_SET> -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl describe rs my-nginx-deployment-9cbcd46b4 -n my-nginx-namespace

Name:           my-nginx-deployment-9cbcd46b4
Namespace:      my-nginx-namespace
Selector:       app=my-nginx-deployment,pod-template-hash=9cbcd46b4
Labels:         app=my-nginx-deployment
                pod-template-hash=9cbcd46b4
Annotations:    deployment.kubernetes.io/desired-replicas: 3
                deployment.kubernetes.io/max-replicas: 4
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/my-nginx-deployment
Replicas:       3 current / 3 desired
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=my-nginx-deployment
           pod-template-hash=9cbcd46b4
  Containers:
   nginx:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:           <none>
```

```bash
kubectl get all -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl get all -n my-nginx-namespace

NAME                                      READY   STATUS    RESTARTS   AGE
pod/my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          21h
pod/my-nginx-deployment-9cbcd46b4-7nmn8   1/1     Running   0          21h
pod/my-nginx-deployment-9cbcd46b4-xt89j   1/1     Running   0          21h

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP   22m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx-deployment   3/3     3            3           21h

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-deployment-9cbcd46b4   3         3         3       21h
```

```bash
kubectl scale --help
```

**Sample Output:**
```bash
$ kubectl scale --help

Set a new size for a deployment, replica set, replication controller, or stateful set.

 Scale also allows users to specify one or more preconditions for the scale action.

 If --current-replicas or --resource-version is specified, it is validated before the scale is attempted, and it is
guaranteed that the precondition holds true when the scale is sent to the server.

Examples:
  # Scale a replica set named 'foo' to 3
  kubectl scale --replicas=3 rs/foo
  
  # Scale a resource identified by type and name specified in "foo.yaml" to 3
  kubectl scale --replicas=3 -f foo.yaml
  
  # If the deployment named mysql's current size is 2, scale mysql to 3
  kubectl scale --current-replicas=2 --replicas=3 deployment/mysql
  
  # Scale multiple replication controllers
  kubectl scale --replicas=5 rc/foo rc/bar rc/baz
  
  # Scale stateful set named 'web' to 3
  kubectl scale --replicas=3 statefulset/web

Options:
    --all=false:
	Select all resources in the namespace of the specified resource types

    --allow-missing-template-keys=true:
	If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to
	golang and jsonpath output formats.

    --current-replicas=-1:
	Precondition for current size. Requires that the current size of the resource match this value in order to
	scale. -1 (default) for no condition.

    --dry-run='none':
	Must be "none", "server", or "client". If client strategy, only print the object that would be sent, without
	sending it. If server strategy, submit server-side request without persisting the resource.

    -f, --filename=[]:
	Filename, directory, or URL to files identifying the resource to set a new size

    -k, --kustomize='':
	Process the kustomization directory. This flag can't be used together with -f or -R.

    -o, --output='':
	Output format. One of: (json, yaml, name, go-template, go-template-file, template, templatefile, jsonpath,
	jsonpath-as-json, jsonpath-file).

    -R, --recursive=false:
	Process the directory used in -f, --filename recursively. Useful when you want to manage related manifests
	organized within the same directory.

    --replicas=0:
	The new desired number of replicas. Required.

    --resource-version='':
	Precondition for resource version. Requires that the current resource version match this value in order to
	scale.

    -l, --selector='':
	Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2). Matching
	objects must satisfy all of the specified label constraints.

    --show-managed-fields=false:
	If true, keep the managedFields when printing objects in JSON or YAML format.

    --template='':
	Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format
	is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

    --timeout=0s:
	The length of time to wait before giving up on a scale operation, zero means don't wait. Any other values
	should contain a corresponding time unit (e.g. 1s, 2m, 3h).

Usage:
  kubectl scale [--resource-version=version] [--current-replicas=count] --replicas=COUNT (-f FILENAME | TYPE NAME)
[options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```


```bash
kubectl scale --replicas=10 rs <YOUR_REPLICA_SET>
```

**Sample Output:**
```bash
$ kubectl get rs -n my-nginx-namespace
NAME                            DESIRED   CURRENT   READY   AGE
my-nginx-deployment-9cbcd46b4   3         3         3       21h
```

Why isn't the ReplicaSet scaling? Because the Deployment manages the ReplicaSet. We need to scale the Deployment, not the ReplicaSet.


```bash
kubectl scale --replicas=10 deployment my-nginx-deployment -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl scale --replicas=10 deployment my-nginx-deployment -n my-nginx-namespace

deployment.apps/my-nginx-deployment scaled
```

```bash
kubectl get all -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl get all -n my-nginx-namespace

NAME                                      READY   STATUS    RESTARTS   AGE
pod/my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          21h
pod/my-nginx-deployment-9cbcd46b4-4hwxb   1/1     Running   0          5m22s
pod/my-nginx-deployment-9cbcd46b4-7fdwm   1/1     Running   0          29s
pod/my-nginx-deployment-9cbcd46b4-8hczb   1/1     Running   0          29s
pod/my-nginx-deployment-9cbcd46b4-dv572   1/1     Running   0          29s
pod/my-nginx-deployment-9cbcd46b4-lgvk7   1/1     Running   0          29s
pod/my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running   0          29s
pod/my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running   0          29s
pod/my-nginx-deployment-9cbcd46b4-v9d9w   1/1     Running   0          29s
pod/my-nginx-deployment-9cbcd46b4-xt89j   1/1     Running   0          21h

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP   36m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx-deployment   10/10   10           10          21h

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-deployment-9cbcd46b4   10        10        10      21h
```

```bash
kubectl apply -f my-nginx.deployment.yaml
```

**Sample Output:**
```bash
$ kubectl apply -f my-nginx-deployment.yaml

deployment.apps/my-nginx-deployment configured
```

```bash
kubectl get all -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl get all -n my-nginx-namespace

NAME                                      READY   STATUS        RESTARTS   AGE
pod/my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running       0          21h
pod/my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running       0          117s
pod/my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running       0          117s

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP   38m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx-deployment   3/3     3            3           21h

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-deployment-9cbcd46b4   3         3         3       21h
```

```bash
grep -i replica my-nginx-deployment.yaml
```

**Sample Output:**
```bash
$ grep -i replica my-nginx-deployment.yaml

  replicas: 3
```


```bash
sed -i 's/replicas\:\ 3/replicas\:\ 10/g' my-nginx-deployment.yaml
```

```bash
grep -i replica my-nginx-deployment.yaml
```

**Sample Output:**
```bash
$ grep -i replica my-nginx-deployment.yaml

  replicas: 10
```


```bash
kubectl apply -f my-nginx-deployment.yaml
```

**Sample Output:**
```bash
$ kubectl apply -f my-nginx-deployment.yaml

deployment.apps/my-nginx-deployment configured
```

```bash
kubectl get all -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl get all -n my-nginx-namespace

NAME                                      READY   STATUS    RESTARTS   AGE
pod/my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          21h
pod/my-nginx-deployment-9cbcd46b4-6hlwl   1/1     Running   0          23s
pod/my-nginx-deployment-9cbcd46b4-7t2fr   1/1     Running   0          23s
pod/my-nginx-deployment-9cbcd46b4-bls2n   1/1     Running   0          23s
pod/my-nginx-deployment-9cbcd46b4-f4lkn   1/1     Running   0          23s
pod/my-nginx-deployment-9cbcd46b4-mxxbl   1/1     Running   0          23s
pod/my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running   0          8m8s
pod/my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running   0          8m8s
pod/my-nginx-deployment-9cbcd46b4-sz27h   1/1     Running   0          23s
pod/my-nginx-deployment-9cbcd46b4-v8bp5   1/1     Running   0          23s

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP   44m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx-deployment   10/10   10           10          21h

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-deployment-9cbcd46b4   10        10        10      21h
```

Checking the `my-nginx-service`, using `curl`:
```bash
curl http://10.111.212.138:8888
```

**Sample Output:**
```bash
$ curl http://10.111.212.138:8888

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



***Transition***

## Adding Labels to Kubernetes Objects Using the `kubectl label` Command

[Kubernetes: Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)



Labels and Selectors play an important role in Kubernetes, so be sure to read the [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) documentation and know the things of value contained within.



Show labels on pods in the `my-nginx-namespace` namespace:
```bash
kubectl get pods -n my-nginx-namespace --show-labels
```

**Sample Output:**
```bash
$ kubectl get pods -n my-nginx-namespace --show-labels
NAME                                  READY   STATUS    RESTARTS   AGE   LABELS
my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          22h   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-6hlwl   1/1     Running   0          81m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-7t2fr   1/1     Running   0          81m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-bls2n   1/1     Running   0          81m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-f4lkn   1/1     Running   0          81m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-mxxbl   1/1     Running   0          81m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running   0          89m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running   0          89m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-sz27h   1/1     Running   0          81m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-v8bp5   1/1     Running   0          81m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
```

kubectl label --help
```

**Sample Output:**
```bash
$ kubectl label --help

Update the labels on a resource.

  *  A label key and value must begin with a letter or number, and may contain letters, numbers, hyphens, dots, and
underscores, up to 63 characters each.
  *  Optionally, the key can begin with a DNS subdomain prefix and a single '/', like example.com/my-app.
  *  If --overwrite is true, then existing labels can be overwritten, otherwise attempting to overwrite a label will
result in an error.
  *  If --resource-version is specified, then updates will use this resource version, otherwise the existing
resource-version will be used.

Examples:
  # Update pod 'foo' with the label 'unhealthy' and the value 'true'
  kubectl label pods foo unhealthy=true

  # Update pod 'foo' with the label 'status' and the value 'unhealthy', overwriting any existing value
  kubectl label --overwrite pods foo status=unhealthy

  # Update all pods in the namespace
  kubectl label pods --all status=unhealthy

  # Update a pod identified by the type and name in "pod.json"
  kubectl label -f pod.json status=unhealthy

  # Update pod 'foo' only if the resource is unchanged from version 1
  kubectl label pods foo status=unhealthy --resource-version=1

  # Update pod 'foo' by removing a label named 'bar' if it exists
  # Does not require the --overwrite flag
  kubectl label pods foo bar-

Options:
    --all=false:
	Select all resources, in the namespace of the specified resource types

    -A, --all-namespaces=false:
	If true, check the specified action in all namespaces.

    --allow-missing-template-keys=true:
	If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to
	golang and jsonpath output formats.

    --dry-run='none':
	Must be "none", "server", or "client". If client strategy, only print the object that would be sent, without
	sending it. If server strategy, submit server-side request without persisting the resource.

    --field-manager='kubectl-label':
	Name of the manager used to track field ownership.

    --field-selector='':
	Selector (field query) to filter on, supports '=', '==', and '!='.(e.g. --field-selector
	key1=value1,key2=value2). The server only supports a limited number of field queries per type.

    -f, --filename=[]:
	Filename, directory, or URL to files identifying the resource to update the labels

    -k, --kustomize='':
	Process the kustomization directory. This flag can't be used together with -f or -R.

    --list=false:
	If true, display the labels for a given resource.

    --local=false:
	If true, label will NOT contact api-server but run locally.

    -o, --output='':
	Output format. One of: (json, yaml, name, go-template, go-template-file, template, templatefile, jsonpath,
	jsonpath-as-json, jsonpath-file).

    --overwrite=false:
	If true, allow labels to be overwritten, otherwise reject label updates that overwrite existing labels.

    -R, --recursive=false:
	Process the directory used in -f, --filename recursively. Useful when you want to manage related manifests
	organized within the same directory.

    --resource-version='':
	If non-empty, the labels update will only succeed if this is the current resource-version for the object. Only
	valid when specifying a single resource.

    -l, --selector='':
	Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2). Matching
	objects must satisfy all of the specified label constraints.

    --show-managed-fields=false:
	If true, keep the managedFields when printing objects in JSON or YAML format.

    --template='':
	Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format
	is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

Usage:
  kubectl label [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]
[options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

```bash
kubectl label pods --all type=webserver -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl label pods --all type=webserver -n my-nginx-namespace

pod/my-nginx-deployment-9cbcd46b4-42n2q labeled
pod/my-nginx-deployment-9cbcd46b4-6hlwl labeled
pod/my-nginx-deployment-9cbcd46b4-7t2fr labeled
pod/my-nginx-deployment-9cbcd46b4-bls2n labeled
pod/my-nginx-deployment-9cbcd46b4-f4lkn labeled
pod/my-nginx-deployment-9cbcd46b4-mxxbl labeled
pod/my-nginx-deployment-9cbcd46b4-nq7xw labeled
pod/my-nginx-deployment-9cbcd46b4-smcv6 labeled
pod/my-nginx-deployment-9cbcd46b4-sz27h labeled
pod/my-nginx-deployment-9cbcd46b4-v8bp5 labeled
```

```bash
kubectl get pods -n my-nginx-namespace --show-labels
```

**Sample Output:**
```bash
$ kubectl get pods -n my-nginx-namespace --show-labels

NAME                                  READY   STATUS    RESTARTS   AGE    LABELS
my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          23h    app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
my-nginx-deployment-9cbcd46b4-6hlwl   1/1     Running   0          98m    app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
my-nginx-deployment-9cbcd46b4-7t2fr   1/1     Running   0          98m    app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
my-nginx-deployment-9cbcd46b4-bls2n   1/1     Running   0          98m    app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
my-nginx-deployment-9cbcd46b4-f4lkn   1/1     Running   0          98m    app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
my-nginx-deployment-9cbcd46b4-mxxbl   1/1     Running   0          98m    app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running   0          106m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running   0          106m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
my-nginx-deployment-9cbcd46b4-sz27h   1/1     Running   0          98m    app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
my-nginx-deployment-9cbcd46b4-v8bp5   1/1     Running   0          98m    app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
```

```bash
kubectl label pods --all type- -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl label pods --all type- -n my-nginx-namespace

pod/my-nginx-deployment-9cbcd46b4-42n2q unlabeled
pod/my-nginx-deployment-9cbcd46b4-6hlwl unlabeled
pod/my-nginx-deployment-9cbcd46b4-7t2fr unlabeled
pod/my-nginx-deployment-9cbcd46b4-bls2n unlabeled
pod/my-nginx-deployment-9cbcd46b4-f4lkn unlabeled
pod/my-nginx-deployment-9cbcd46b4-mxxbl unlabeled
pod/my-nginx-deployment-9cbcd46b4-nq7xw unlabeled
pod/my-nginx-deployment-9cbcd46b4-smcv6 unlabeled
pod/my-nginx-deployment-9cbcd46b4-sz27h unlabeled
pod/my-nginx-deployment-9cbcd46b4-v8bp5 unlabeled
```

```bash
kubectl get pods -n my-nginx-namespace --show-labels
```

**Sample Output:**
```bash
$ kubectl get pods -n my-nginx-namespace --show-labels

NAME                                  READY   STATUS    RESTARTS   AGE    LABELS
my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          23h    app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-6hlwl   1/1     Running   0          102m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-7t2fr   1/1     Running   0          102m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-bls2n   1/1     Running   0          102m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-f4lkn   1/1     Running   0          102m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-mxxbl   1/1     Running   0          102m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running   0          110m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running   0          110m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-sz27h   1/1     Running   0          102m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
my-nginx-deployment-9cbcd46b4-v8bp5   1/1     Running   0          102m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
```

```bash
kubectl get all -n my-nginx-namespace --show-labels
```

**Sample Output:**
```bash
$ kubectl get all -n my-nginx-namespace --show-labels

NAME                                      READY   STATUS    RESTARTS   AGE    LABELS
pod/my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          23h    app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-6hlwl   1/1     Running   0          105m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-7t2fr   1/1     Running   0          105m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-bls2n   1/1     Running   0          105m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-f4lkn   1/1     Running   0          105m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-mxxbl   1/1     Running   0          105m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running   0          113m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running   0          113m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-sz27h   1/1     Running   0          105m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-v8bp5   1/1     Running   0          105m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE    LABELS
service/my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP   149m   app=my-nginx-service

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
deployment.apps/my-nginx-deployment   10/10   10           10          23h   app=my-nginx-deployment

NAME                                            DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/my-nginx-deployment-9cbcd46b4   10        10        10      23h   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
```

```bash
kubectl label deploy my-nginx-deployment type=webserver -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl label deploy my-nginx-deployment type=webserver -n my-nginx-namespace

deployment.apps/my-nginx-deployment labeled
```

```bash
kubectl get all -n my-nginx-namespace --show-labels
```

**Sample Output:**
```bash
$ kubectl get all -n my-nginx-namespace --show-labels

NAME                                      READY   STATUS    RESTARTS   AGE    LABELS
pod/my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          23h    app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-6hlwl   1/1     Running   0          107m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-7t2fr   1/1     Running   0          107m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-bls2n   1/1     Running   0          107m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-f4lkn   1/1     Running   0          107m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-mxxbl   1/1     Running   0          107m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running   0          115m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running   0          115m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-sz27h   1/1     Running   0          107m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-v8bp5   1/1     Running   0          107m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE    LABELS
service/my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP   151m   app=my-nginx-service

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
deployment.apps/my-nginx-deployment   10/10   10           10          23h   app=my-nginx-deployment,type=webserver

NAME                                            DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/my-nginx-deployment-9cbcd46b4   10        10        10      23h   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
```

```bash
kubectl label pods,rs --all type=webserver -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl label pods,rs --all type=webserver -n my-nginx-namespace

pod/my-nginx-deployment-9cbcd46b4-42n2q labeled
pod/my-nginx-deployment-9cbcd46b4-6hlwl labeled
pod/my-nginx-deployment-9cbcd46b4-7t2fr labeled
pod/my-nginx-deployment-9cbcd46b4-bls2n labeled
pod/my-nginx-deployment-9cbcd46b4-f4lkn labeled
pod/my-nginx-deployment-9cbcd46b4-mxxbl labeled
pod/my-nginx-deployment-9cbcd46b4-nq7xw labeled
pod/my-nginx-deployment-9cbcd46b4-smcv6 labeled
pod/my-nginx-deployment-9cbcd46b4-sz27h labeled
pod/my-nginx-deployment-9cbcd46b4-v8bp5 labeled
replicaset.apps/my-nginx-deployment-9cbcd46b4 labeled
```

```bash
kubectl get all -n my-nginx-namespace --show-labels
```

**Sample Output:**
```bash
$ kubectl get all -n my-nginx-namespace --show-labels
NAME                                      READY   STATUS    RESTARTS   AGE    LABELS
pod/my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          23h    app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
pod/my-nginx-deployment-9cbcd46b4-6hlwl   1/1     Running   0          110m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
pod/my-nginx-deployment-9cbcd46b4-7t2fr   1/1     Running   0          110m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
pod/my-nginx-deployment-9cbcd46b4-bls2n   1/1     Running   0          110m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
pod/my-nginx-deployment-9cbcd46b4-f4lkn   1/1     Running   0          110m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
pod/my-nginx-deployment-9cbcd46b4-mxxbl   1/1     Running   0          110m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
pod/my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running   0          117m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
pod/my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running   0          117m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
pod/my-nginx-deployment-9cbcd46b4-sz27h   1/1     Running   0          110m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
pod/my-nginx-deployment-9cbcd46b4-v8bp5   1/1     Running   0          110m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE    LABELS
service/my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP   154m   app=my-nginx-service

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
deployment.apps/my-nginx-deployment   10/10   10           10          23h   app=my-nginx-deployment,type=webserver

NAME                                            DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/my-nginx-deployment-9cbcd46b4   10        10        10      23h   app=my-nginx-deployment,pod-template-hash=9cbcd46b4,type=webserver
```

```bash
kubectl label pods,rs,deploy --all type- -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl label pods,rs,deploy --all type- -n my-nginx-namespace

pod/my-nginx-deployment-9cbcd46b4-42n2q unlabeled
pod/my-nginx-deployment-9cbcd46b4-6hlwl unlabeled
pod/my-nginx-deployment-9cbcd46b4-7t2fr unlabeled
pod/my-nginx-deployment-9cbcd46b4-bls2n unlabeled
pod/my-nginx-deployment-9cbcd46b4-f4lkn unlabeled
pod/my-nginx-deployment-9cbcd46b4-mxxbl unlabeled
pod/my-nginx-deployment-9cbcd46b4-nq7xw unlabeled
pod/my-nginx-deployment-9cbcd46b4-smcv6 unlabeled
pod/my-nginx-deployment-9cbcd46b4-sz27h unlabeled
pod/my-nginx-deployment-9cbcd46b4-v8bp5 unlabeled
replicaset.apps/my-nginx-deployment-9cbcd46b4 unlabeled
deployment.apps/my-nginx-deployment unlabeled
```

```bash
kubectl get all -n my-nginx-namespace --show-labels
```

**Sample Output:**
```bash
$ kubectl get all -n my-nginx-namespace --show-labels

NAME                                      READY   STATUS    RESTARTS   AGE    LABELS
pod/my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          23h    app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-6hlwl   1/1     Running   0          112m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-7t2fr   1/1     Running   0          112m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-bls2n   1/1     Running   0          112m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-f4lkn   1/1     Running   0          112m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-mxxbl   1/1     Running   0          112m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running   0          120m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running   0          120m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-sz27h   1/1     Running   0          112m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
pod/my-nginx-deployment-9cbcd46b4-v8bp5   1/1     Running   0          112m   app=my-nginx-deployment,pod-template-hash=9cbcd46b4

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE    LABELS
service/my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP   156m   app=my-nginx-service

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
deployment.apps/my-nginx-deployment   10/10   10           10          23h   app=my-nginx-deployment

NAME                                            DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/my-nginx-deployment-9cbcd46b4   10        10        10      23h   app=my-nginx-deployment,pod-template-hash=9cbcd46b4
```

```bash
kubectl get nodes --show-labels
```

**Sample Output:**
```bash
$ kubectl get nodes --show-labels

NAME               STATUS   ROLES           AGE   VERSION   LABELS
control-plane-01   Ready    control-plane   8d    v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=control-plane-01,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
worker-node-01     Ready    <none>          8d    v1.26.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker-node-01,kubernetes.io/os=linux
```



Again, labels and Selectors play an important role in Kubernetes, so be sure to read the [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) documentation and know the things of value contained within.

***Transition***

## Checking Logs Using `kubectl`



```bash
kubectl logs --help
```

**Sample Output:**
```bash
$ kubectl logs --help

Print the logs for a container in a pod or specified resource. If the pod has only one container, the container name is
optional.

Examples:
  # Return snapshot logs from pod nginx with only one container
  kubectl logs nginx

  # Return snapshot logs from pod nginx with multi containers
  kubectl logs nginx --all-containers=true

  # Return snapshot logs from all containers in pods defined by label app=nginx
  kubectl logs -l app=nginx --all-containers=true

  # Return snapshot of previous terminated ruby container logs from pod web-1
  kubectl logs -p -c ruby web-1

  # Begin streaming the logs of the ruby container in pod web-1
  kubectl logs -f -c ruby web-1

  # Begin streaming the logs from all containers in pods defined by label app=nginx
  kubectl logs -f -l app=nginx --all-containers=true

  # Display only the most recent 20 lines of output in pod nginx
  kubectl logs --tail=20 nginx

  # Show all logs from pod nginx written in the last hour
  kubectl logs --since=1h nginx

  # Show logs from a kubelet with an expired serving certificate
  kubectl logs --insecure-skip-tls-verify-backend nginx

  # Return snapshot logs from first container of a job named hello
  kubectl logs job/hello

  # Return snapshot logs from container nginx-1 of a deployment named nginx
  kubectl logs deployment/nginx -c nginx-1

Options:
    --all-containers=false:
	Get all containers' logs in the pod(s).

    -c, --container='':
	Print the logs of this container

    -f, --follow=false:
	Specify if the logs should be streamed.

    --ignore-errors=false:
	If watching / following pod logs, allow for any errors that occur to be non-fatal

    --insecure-skip-tls-verify-backend=false:
	Skip verifying the identity of the kubelet that logs are requested from.  In theory, an attacker could provide
	invalid log content back. You might want to use this if your kubelet serving certificates have expired.

    --limit-bytes=0:
	Maximum bytes of logs to return. Defaults to no limit.

    --max-log-requests=5:
	Specify maximum number of concurrent logs to follow when using by a selector. Defaults to 5.

    --pod-running-timeout=20s:
	The length of time (like 5s, 2m, or 3h, higher than zero) to wait until at least one pod is running

    --prefix=false:
	Prefix each log line with the log source (pod name and container name)

    -p, --previous=false:
	If true, print the logs for the previous instance of the container in a pod if it exists.

    -l, --selector='':
	Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2). Matching
	objects must satisfy all of the specified label constraints.

    --since=0s:
	Only return logs newer than a relative duration like 5s, 2m, or 3h. Defaults to all logs. Only one of
	since-time / since may be used.

    --since-time='':
	Only return logs after a specific date (RFC3339). Defaults to all logs. Only one of since-time / since may be
	used.

    --tail=-1:
	Lines of recent log file to display. Defaults to -1 with no selector, showing all log lines otherwise 10, if a
	selector is provided.

    --timestamps=false:
	Include timestamps on each line in the log output

Usage:
  kubectl logs [-f] [-p] (POD | TYPE/NAME) [-c CONTAINER] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

```bash
kubectl logs my-nginx-deployment-9cbcd46b4-42n2q -n my-nginx-namespace
```

```bash
kubectl get pods -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl get pods -n my-nginx-namespace
NAME                                  READY   STATUS    RESTARTS   AGE
my-nginx-deployment-9cbcd46b4-42n2q   1/1     Running   0          24h
my-nginx-deployment-9cbcd46b4-6hlwl   1/1     Running   0          156m
my-nginx-deployment-9cbcd46b4-7t2fr   1/1     Running   0          156m
my-nginx-deployment-9cbcd46b4-bls2n   1/1     Running   0          156m
my-nginx-deployment-9cbcd46b4-f4lkn   1/1     Running   0          156m
my-nginx-deployment-9cbcd46b4-mxxbl   1/1     Running   0          156m
my-nginx-deployment-9cbcd46b4-nq7xw   1/1     Running   0          164m
my-nginx-deployment-9cbcd46b4-smcv6   1/1     Running   0          164m
my-nginx-deployment-9cbcd46b4-sz27h   1/1     Running   0          156m
my-nginx-deployment-9cbcd46b4-v8bp5   1/1     Running   0          156m
```

```bash
kubectl logs my-nginx-deployment-9cbcd46b4-42n2q -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl logs my-nginx-deployment-9cbcd46b4-42n2q -n my-nginx-namespace

/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/04/27 19:19:20 [notice] 1#1: using the "epoll" event method
2023/04/27 19:19:20 [notice] 1#1: nginx/1.23.4
2023/04/27 19:19:20 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6) 
2023/04/27 19:19:20 [notice] 1#1: OS: Linux 5.15.0-69-generic
2023/04/27 19:19:20 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/04/27 19:19:20 [notice] 1#1: start worker processes
2023/04/27 19:19:20 [notice] 1#1: start worker process 29
2023/04/27 19:19:20 [notice] 1#1: start worker process 30
10.0.1.132 - - [27/Apr/2023:19:25:37 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.81.0" "-"
10.0.1.132 - - [28/Apr/2023:16:01:12 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.81.0" "-"
```

```bash
kubectl logs -f -l app=my-nginx-deployment --all-containers=true -n my-nginx-namespace --max-log-requests 10
```



***Transition***

## Exporting Running Kubernetes Objects to a Manifest Using the `kubectl get` Command



```bash
kubectl get deploy my-nginx-deployment -n my-nginx-namespace -o yaml
```

**Sample Output:**
```bash
$ kubectl get deploy my-nginx-deployment -n my-nginx-namespace -o yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"creationTimestamp":null,"labels":{"app":"my-nginx-deployment"},"name":"my-nginx-deployment","namespace":"my-nginx-namespace"},"spec":{"replicas":10,"selector":{"matchLabels":{"app":"my-nginx-deployment"}},"strategy":{},"template":{"metadata":{"creationTimestamp":null,"labels":{"app":"my-nginx-deployment"}},"spec":{"containers":[{"image":"nginx:latest","name":"nginx","ports":[{"containerPort":80}],"resources":{}}]}}},"status":{}}
  creationTimestamp: "2023-04-27T19:19:11Z"
  generation: 6
  labels:
    app: my-nginx-deployment
  name: my-nginx-deployment
  namespace: my-nginx-namespace
  resourceVersion: "979655"
  uid: 26df08d5-4b69-41a5-acc8-d444fc4239bd
spec:
  progressDeadlineSeconds: 600
  replicas: 10
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: my-nginx-deployment
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: my-nginx-deployment
    spec:
      containers:
      - image: nginx:latest
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 10
  conditions:
  - lastTransitionTime: "2023-04-27T19:19:11Z"
    lastUpdateTime: "2023-04-27T19:19:20Z"
    message: ReplicaSet "my-nginx-deployment-9cbcd46b4" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2023-04-28T16:44:39Z"
    lastUpdateTime: "2023-04-28T16:44:39Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  observedGeneration: 6
  readyReplicas: 10
  replicas: 10
  updatedReplicas: 10
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
  replicas: 10
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

## Deleting Kubernetes Objects Using `kubectl delete`



```bash
kubectl get all -A
```

**Sample Output:**
```bash
$ kubectl get all -A
NAMESPACE            NAME                                           READY   STATUS    RESTARTS   AGE
declarative          pod/nginx-pod                                  1/1     Running   0          26h
imperative           pod/nginx-pod                                  1/1     Running   0          35s
kube-system          pod/calico-kube-controllers-57b57c56f-4hvfl    1/1     Running   0          8d
kube-system          pod/calico-node-s28hb                          1/1     Running   0          8d
kube-system          pod/calico-node-tqb6g                          1/1     Running   0          8d
kube-system          pod/coredns-787d4945fb-cg5wh                   1/1     Running   0          8d
kube-system          pod/coredns-787d4945fb-xdbll                   1/1     Running   0          8d
kube-system          pod/etcd-control-plane-01                      1/1     Running   0          8d
kube-system          pod/kube-apiserver-control-plane-01            1/1     Running   0          8d
kube-system          pod/kube-controller-manager-control-plane-01   1/1     Running   0          8d
kube-system          pod/kube-proxy-blgcf                           1/1     Running   0          8d
kube-system          pod/kube-proxy-t9wb4                           1/1     Running   0          8d
kube-system          pod/kube-scheduler-control-plane-01            1/1     Running   0          8d
my-nginx-namespace   pod/my-nginx-deployment-9cbcd46b4-42n2q        1/1     Running   0          26h
my-nginx-namespace   pod/my-nginx-deployment-9cbcd46b4-6hlwl        1/1     Running   0          4h50m
my-nginx-namespace   pod/my-nginx-deployment-9cbcd46b4-7t2fr        1/1     Running   0          4h50m
my-nginx-namespace   pod/my-nginx-deployment-9cbcd46b4-bls2n        1/1     Running   0          4h50m
my-nginx-namespace   pod/my-nginx-deployment-9cbcd46b4-f4lkn        1/1     Running   0          4h50m
my-nginx-namespace   pod/my-nginx-deployment-9cbcd46b4-mxxbl        1/1     Running   0          4h50m
my-nginx-namespace   pod/my-nginx-deployment-9cbcd46b4-nq7xw        1/1     Running   0          4h58m
my-nginx-namespace   pod/my-nginx-deployment-9cbcd46b4-smcv6        1/1     Running   0          4h58m
my-nginx-namespace   pod/my-nginx-deployment-9cbcd46b4-sz27h        1/1     Running   0          4h50m
my-nginx-namespace   pod/my-nginx-deployment-9cbcd46b4-v8bp5        1/1     Running   0          4h50m

NAMESPACE            NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default              service/kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP                  8d
kube-system          service/kube-dns           ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   8d
my-nginx-namespace   service/my-nginx-service   ClusterIP   10.111.212.138   <none>        8888/TCP                 5h34m

NAMESPACE     NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/calico-node   2         2         2       2            2           kubernetes.io/os=linux   8d
kube-system   daemonset.apps/kube-proxy    2         2         2       2            2           kubernetes.io/os=linux   8d

NAMESPACE            NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system          deployment.apps/calico-kube-controllers   1/1     1            1           8d
kube-system          deployment.apps/coredns                   2/2     2            2           8d
my-nginx-namespace   deployment.apps/my-nginx-deployment       10/10   10           10          26h

NAMESPACE            NAME                                                DESIRED   CURRENT   READY   AGE
kube-system          replicaset.apps/calico-kube-controllers-57b57c56f   1         1         1       8d
kube-system          replicaset.apps/coredns-787d4945fb                  2         2         2       8d
my-nginx-namespace   replicaset.apps/my-nginx-deployment-9cbcd46b4       10        10        10      26h
```

```bash
kubectl delete pod/nginx-pod -n imperative
```

**Sample Output:**
```bash
$ kubectl delete pod/nginx-pod -n imperative

pod "nginx-pod" deleted
```

```bash
kubectl delete ns/imperative
```

**Sample Output:**
```bash
$ kubectl delete ns/imperative

namespace "imperative" deleted
```

```bash
kubectl get all -n imperative
```

**Sample Output:**
```bash
$ kubectl get all -n imperative

No resources found in imperative namespace.
```


```bash
kubectl delete pod/nginx-pod -n declarative ; kubectl delete ns/declarative
```

**Sample Output:**
```bash
$ kubectl delete pod/nginx-pod -n declarative ; kubectl delete ns/declarative

pod "nginx-pod" deleted
namespace "declarative" deleted
```



```bash
kubectl delete -f my-nginx-service.yaml
```

```bash
kubectl delete -f my-nginx-deployment.yaml
```

```bash
kubectl delete -f my-nginx-namespace.yaml
```

```bash
kubectl get all -n my-nginx-namespace
```

**Sample Output:**
```bash
$ kubectl delete -f my-nginx-service.yaml

service "my-nginx-service" deleted

$ kubectl delete -f my-nginx-deployment.yaml

deployment.apps "my-nginx-deployment" deleted

$ kubectl delete -f my-nginx-namespace.yaml

namespace "my-nginx-namespace" deleted

$ kubectl get all -n my-nginx-namespace

No resources found in my-nginx-namespace namespace.
```


```bash
kubectl get all -A
```

**Sample Output:**
```bash
$ kubectl get all -A

NAMESPACE     NAME                                           READY   STATUS    RESTARTS   AGE
kube-system   pod/calico-kube-controllers-57b57c56f-4hvfl    1/1     Running   0          8d
kube-system   pod/calico-node-s28hb                          1/1     Running   0          8d
kube-system   pod/calico-node-tqb6g                          1/1     Running   0          8d
kube-system   pod/coredns-787d4945fb-cg5wh                   1/1     Running   0          8d
kube-system   pod/coredns-787d4945fb-xdbll                   1/1     Running   0          8d
kube-system   pod/etcd-control-plane-01                      1/1     Running   0          8d
kube-system   pod/kube-apiserver-control-plane-01            1/1     Running   0          8d
kube-system   pod/kube-controller-manager-control-plane-01   1/1     Running   0          8d
kube-system   pod/kube-proxy-blgcf                           1/1     Running   0          8d
kube-system   pod/kube-proxy-t9wb4                           1/1     Running   0          8d
kube-system   pod/kube-scheduler-control-plane-01            1/1     Running   0          8d

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  8d
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   8d

NAMESPACE     NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/calico-node   2         2         2       2            2           kubernetes.io/os=linux   8d
kube-system   daemonset.apps/kube-proxy    2         2         2       2            2           kubernetes.io/os=linux   8d

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           8d
kube-system   deployment.apps/coredns                   2/2     2            2           8d

NAMESPACE     NAME                                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/calico-kube-controllers-57b57c56f   1         1         1       8d
kube-system   replicaset.apps/coredns-787d4945fb                  2         2         2       8d
```

***Everything you created has been cleaned up. Good job!***

## Practice Exercises

Use the following practice exercises to test the skills you learned in this tutorial. Try and solve the exercises on your own before checking the solutions.

### Exercise 1

***Create a single Pod, imperatively, named `apache-pod`, using the image `httpd:latest`, in the `apache-imperative` namespace, with `port 80` exposed. Confirm that your Pod is in the `Running` state. Confirm you can reach the Apache web server using `curl`.***

[Exercise 1: Solution](solutions/solution-exercise-1.md)

### Exercise 2

***Create a YAML manifest for a single Pod, imperatively, named `apache-pod`, using the image `httpd:latest`, in the `apache-declarative` namespace, with `port 80` exposed. Review this manifest, deploy it, and confirm all the objects have been created. Confirm that your Pod is in the `Running` state. Confirm you can reach the Apache web server using `curl`.***

[Exercise 2: Solution](solutions/solution-exercise-2.md)

### Exercise 3

***Create a YAML manifest for a Namespace, named `my-nginx-namespace` in a file named `my-nginx-namespace.yaml`. Review this manifest, deploy it, and confirm all the objects have been created.***

[Exercise 3: Solution](solutions/solution-exercise-3.md)

### Exercise 4

***Create a YAML manifest for a Deployment, named `my-nginx-deployment`, using the `nginx-latest` container image, in the `my-nginx-namespace` namespace, with 3 replicas, exposing port 80, in a file named `my-nginx-deployment.yaml`. Review this manifest, deploy it, and confirm all the objects have been created.***

[Exercise 4: Solution](solutions/solution-exercise-4.md)

### Exercise 5



[Exercise 5: Solution](solutions/solution-exercise-5.md)

***Transition***

## Summary



Enjoy!

*Tom Dean*
