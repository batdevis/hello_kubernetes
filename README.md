
# Hello, kubernetes
I'll try to focus only on the essentials steps in order to practice in a kubernetes sandbox local cluster. 
I want to keep things essentials and simple, without going deeper and omitting non-essential informations.

## Prepare your machine with Minikube
We use **Minikube**, the local Kubernetes cluster system:
[https://minikube.sigs.k8s.io/docs/overview/](https://minikube.sigs.k8s.io/docs/overview/).

Minikube creates a kubernetes cluster inside a **virtual machine**.

Follow the instructions:
[https://minikube.sigs.k8s.io/docs/start/linux/](https://minikube.sigs.k8s.io/docs/start/linux/).

*I work in linux. Don't ask me about other operating systems*.

Start minikube:
```
$ minikube start
```

Open the kubernetes web dashboard and celebrate this first step.
```
$ minikube dashboard
```
![kubernetes dashboard](https://github.com/batdevis/hello_kubernetes/raw/master/images/k8s_minikube_dashboard.png)

Cool, isn't it?
We'll never use the dashboard in this tutorial.
Back to terminal, press ctrl+C and type...

### Kubectl
[Here](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux)'s the instructions to install kubectl cli.
```
$ kubectl get pods -A
```
```
NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
kube-system            coredns-5644d7b6d9-5jxj4                     1/1     Running   0          139m
kube-system            coredns-5644d7b6d9-9xr9c                     1/1     Running   0          139m
...
```
Ok, this is better.
Your local k8s system is up.
If you want to learn to work with kubernetes, you have to learn to use the command line.
The web console is only an extra.
If you think you can manage devops issues only with a web page and a mouse pointer, do the world a favor and change jobs.

## Kubernetes components
Kubernetes has many components.

+ Cluster
    + Nodes
        + Ingress
        + Services
        + Deployments
            + Pods
                   + Container
                   + ReplicaSet
        + Storage
        + ConfigMaps
        + Secrets

In kubernetes every component is rapresented by a **yaml** configuration.

### Cluster
It's the main box that contains all components:
[https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-cluster](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-cluster).
The cluster contains the nodes.

### Nodes
A [node](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-node) typically is a virtual machine.
In a cluster we have at least one **master node** and at least one **worker node**.
[This Google guide](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-architecture)  explains this concepts deeper.

The master nodes have inside all kubernetes things, we won't touch it.
The worker nodes will contain our applications, it's our battle field.

### Pods
Nodes contains the pods.
The [pod](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-pod) is the main component of kubernetes.
You will be always focus on the pods every time you do something with kubectl.
"Pods are used as the unit of replication in Kubernetes." ( [Daniel Sanche](https://link.medium.com/QOXL6B9iC0).
A pod contains a **docker [container](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-container)**.

*I said "usually" because sometimes it could contains more than ine container, and the container could be other than docker, [like rkt](https://containerjournal.com/topics/container-ecosystems/5-container-alternatives-to-docker/).
But we stay on the main road.*

If you manually kill a pod, his deployment will start immediately a new instance.
If you want less pods, you must to decrease the number of replicas.

### Deployment
One Deployment controls the instances his pods.
In the deployment configuration you set the number of **replicas** of the pod: this is one of the most important features of kubernetes.

*deployment.app.yaml*
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 4
...
```
Around them, pods have **services**, **configmaps**, **secrets**, **volumes**.

### Service
A [service](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-service) is a component that maps the ports of the container with the other services.

An example is a rails app that run on port 3000 of the pod, but is mapped in the port 80 of his service:

*service.app.yaml*
```
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: app
``` 
### Configmaps, secrets
In **configmaps** and **secrets** you put configuration key-value pairs and configuration files.
Usually you map this values with env variables of the container.

*configmap.app.yaml*
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app
  namespace: default
data:
  RAILS_ENV: production
```

[Secrets](https://kubernetes.io/docs/reference/glossary/?all=true#term-secret) are like configmaps, but base64 encoded.

### Volumes
Piece of disks where to [store data](https://kubernetes.io/docs/reference/glossary/?all=true#term-volume), mounted in your pods.

### Ingress
It's the [point of connection](https://kubernetes.io/docs/reference/glossary/?all=true#term-ingress) with the external world. Like a load balancer.
Ask to the ingress if you want to know the public ip of your cluster:
```
$ kubectl get ingress
```
## Deploy


