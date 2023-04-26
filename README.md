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

[]()

[]()

[]()

[]()

[]()

[]()


## Prerequisites

[Kubernetes: Install Tools](https://kubernetes.io/docs/tasks/tools/)

You will need...

```bash
kubectl cluster-resources
```

*The examples in this guide come from an Ubuntu Linux 22.04 operating system environment with a Kubernetes 1.26.1 cluster configured with `kubeadm`.*

## Exploring the `kubectl` Command

[Kubernetes: Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/)

[Kubernetes: kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)



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

You should get `pod/nginx-pod created`.  Checking our work:

```bash
$ kubectl get pod --namespace imperative

NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          30s
```

***Transition***

### Step 2



***Transition***

### Step 2



***Transition***

### Step 2



***Transition***

### Step 2



***Transition***

### Step 2



***Transition***

## Summary



Enjoy!

*Tom Dean*
