# Solution: Exercise 1
**Tom Dean - 5/16/2023**

## Introduction

This exercise tests your ability to create both a **Namespace** and a **Pod** imperatively.

## Exercise

***Create a single Pod, imperatively, named `apache-pod`, using the image `httpd:latest`, in the `apache-imperative` namespace, with `port 80` exposed. Confirm that your Pod is in the `Running` state. Confirm you can reach the Apache web server using `curl`.***

## Solution

### Step 1: Create Namespace Imperatively


```bash
kubectl create namespace apache-imperative
```

**Sample Output:**
```bash
$ kubectl create namespace apache-imperative

namespace/apache-imperative created
```


### Step 2: Create Pod Imperatively


```bash
kubectl run apache-pod --image httpd:latest --namespace apache-imperative --port=80
```

**Sample Output:**
```bash
$ kubectl run apache-pod --image httpd:latest --namespace apache-imperative --port=80

pod/apache-pod created
```

### Step 3: Check Your Work



```bash
kubectl get pod --namespace apache-imperative -o wide
```

**Sample Output:**
```bash
$ kubectl get pod --namespace apache-imperative -o wide

NAME         READY   STATUS    RESTARTS   AGE   IP                NODE               NOMINATED NODE   READINESS GATES
apache-pod   1/1     Running   0          42s   192.168.173.210   control-plane-01   <none>           <none>
```

```bash
curl http://<POD_IP>
```

**Sample Output:**
```bash
$ curl http://192.168.173.210

<html><body><h1>It works!</h1></body></html>
```

## Summary


