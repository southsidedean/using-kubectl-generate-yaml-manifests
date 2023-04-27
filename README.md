# Using `kubectl` to Generate YAML Manifests
**Tom Dean - 4/26/2023**

## Introduction

I'm preparing to take the #CKAD and #CKA exams, and one area I need to build my #k8s muscles is using the `kubectl` command imperatively to create #Kubernetes manifests for #declarative deployment.

On the CKAD and CKA exams, every second counts, and this skill facilitates the rapid and consistent generation of #YAML manifests that can quickly be edited and deployed to Kubernetes.

Adrien Trouillaud's excellent Medium article, [Imperative vs. Declarative — a Kubernetes Tutorial](https://medium.com/payscale-tech/imperative-vs-declarative-a-kubernetes-tutorial-4be66c5d8914) while researching this topic. Adrien's article is a comprehensive tutorial on how to leverage the #imperative approach to creating objects in Kubernetes.

In this tutorial, we're going to examine how we can use the `kubectl` command *imperatively* to create some common objects as *declarative* YAML manifests.

***Transition***

## References

[Medium: Imperative vs. Declarative — a Kubernetes Tutorial](https://medium.com/payscale-tech/imperative-vs-declarative-a-kubernetes-tutorial-4be66c5d8914)

[Kubernetes: Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)

[Kubernetes: kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

[Kubernetes: Install Tools](https://kubernetes.io/docs/tasks/tools/)

[GitHub: cka-d-cluster-builder-lab](https://github.com/southsidedean/cka-d-cluster-builder-lab)

[GitHub: Deploying a Kubernetes In Docker (KIND) Cluster Using Podman on Ubuntu Linux](https://github.com/southsidedean/deploy-kind-using-podman-ubuntu)

[]()

[]()

[]()

[]()


## Prerequisites

[Kubernetes: Install Tools](https://kubernetes.io/docs/tasks/tools/)

[GitHub: cka-d-cluster-builder-lab](https://github.com/southsidedean/cka-d-cluster-builder-lab)

[GitHub: Deploying a Kubernetes In Docker (KIND) Cluster Using Podman on Ubuntu Linux](https://github.com/southsidedean/deploy-kind-using-podman-ubuntu)

*The examples in this guide come from an Ubuntu Linux 22.04 operating system environment with a Kubernetes 1.26.1 cluster configured with `kubeadm`.*

You're going to need a Kubernetes environment to perform the steps in this tutorial.  If you have access to one already, you can use it.  If not, you'll need to create one.  I'v provided some resources in the links in this section.

The first link, from the Kubernetes documentation, contains resources detailing several options for cluster installation.

I've also provided links to two of my GitHub repositories with tutorials, one for creating a `kubeadm` cluster and a second for creating a Kubernetes In Docker, or KIND, cluster.  Either of these will also work for this tutorial.

When you have your cluster, test it:
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

***With a working cluster at our disposal, we're ready to proceed.***

## Exploring the `kubectl` Command

[Kubernetes: Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)

[Kubernetes: kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

The primary tool you're going to use to interact with your Kubernetes cluster, both in this tutorial and the CKA/CKAD exams, is `kubectl`.  The `kubectl` command gives us a user-friendly (or user-friendlier) method of interacting with the Kubernetes API.

```bash
kubectl --help
```


```bash
kubectl run --help
```

```bash
kubectl api-resources
```


***Transition***

### Creating Pods Using the `kubectl run` Command



Say you've been presented with the following request:

"Create a *single* pod, named `nginx-pod`, using the image `nginx:latest`, in the `imperative` namespace, with port 80 exposed."

Create the `nginx-pod` imperatively:
```bash
kubectl run nginx-pod --image nginx:latest --namespace imperative --port=80
```

You will get the message `Error from server (NotFound): namespaces "imperative" not found`.  You need to create the namespace.

Create the `imperative` namespace imperatively:
```bash
kubectl create namespace imperative
```

You should see `namespace/imperative created`.

Try it again.  Create the `nginx-pod` imperatively:
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
nginx-pod   1/1     Running   0          44m   192.168.126.200   worker-node-01   <none>           <none>
```


```bash
curl http://<POD_IP>
```

**Sample Output:**
```bash
$ curl http://192.168.126.200

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

### Creating Pod Manifests Using the `kubectl run` Command



***Transition***

### Creating Deployment Manifests Using the `kubectl create` Command



***Transition***

### Creating Service Manifests Using the `kubectl create` Command



***Transition***

### Section



***Transition***

### Section



***Transition***

### Section



***Transition***

### Section



***Transition***

### Section



***Transition***

### Section



***Transition***

### Practice Exercises



***Transition***

## Summary



Enjoy!

*Tom Dean*
