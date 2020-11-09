# Kubernetes Objects
Kubernetes provides several abstractions to make it possible to specify our infrastructure as configuration.
All of these objects are `.yaml` files with keys that Kubernetes uses to maintain our cluster.

## Deployment
A [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is a description of updates for [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).
The Control Plane makes changes to the state of the cluster to the desired state specicified in a Deployment.

The following Deployment creates 3 Pods running the container `nginx:1.14.2` on port `80`. 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```

## Pod
A [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) is the simplest unit of deployment.
It is made of one or more containers that run on the same host, mount the same storage volumes and share the same network namespace. They're assigned an IP address that they can use to communicate with other Pods in the cluster.

For example, the following Pod prints "Hello Kubernetes".
```
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
```

## Labels
[Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors) are key-value pairs attached to Kubernetes objects that Kubernetes uses to organize and select Kubernetes objects.
Labels support equality (`==`, `!=`) and set based (`in`, `notin`, `exists`) operators that controllers combine to manage cluster state.

For example, the ReplicaSet below has the `app` and `tier` labels.
Kubernetes can use this information to apply network routing rules.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```



## ReplicaSets
[ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) are how Kubernetes tracks how many replicas of a pod should be running.
When the state of the Pods falls out of sync with the desired state as specified by the ReplicaSet, the 
Deployment will automatically create or destroy pods to match the desired state.
Deployments update ReplicaSets and Pods.
When a new Deployment is created, a new ReplicaSet will be created and the old ReplicaSet scaled to 0 replicas.
This allows Kubernetes to roll back a Deployment to its previous state.

For example, this ReplicaSet runs 3 instances of the `gcr.io/google_samples/gb-frontend:v3` instance.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```


## Namespace
A [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) is a virtual cluster within a parent cluster.
Kubernetes uses namespaces to isolate applications on a cluster from each other.

These are the default namespaces returned by `kubectl get namespace`.

```
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
```