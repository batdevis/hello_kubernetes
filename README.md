![kubernetes logo](https://github.com/batdevis/hello_kubernetes/raw/master/images/k8s_logo_200.png)

# Hello, kubernetes
## We live in a containerized world.
As developers, we always try to isolate problems, functions, bugs, tasks.

We try to blackbox everyhing and let communicate every component through a minimal set of messages.

With Kubernetes we apply the same concepts to the world of the application deployment.

They call it _Infrastructure as Code_.

Kubernetes is the orchestrator tool for managing containerized applications.

I'll try to focus only on the essentials steps in order to practice in a kubernetes sandbox local cluster.

I want to keep things essentials and simple, without going deeper.

## Prepare your machine with Minikube
![minikube logo](https://github.com/batdevis/hello_kubernetes/raw/master/images/minikube_logo_200.png)

We are going to use **Minikube**, the local Kubernetes cluster system: [https://minikube.sigs.k8s.io/docs/overview/](https://minikube.sigs.k8s.io/docs/overview/).

Minikube creates a kubernetes cluster inside a **virtual machine**.

Follow the instructions to install it on [Virtual Box](https://www.virtualbox.org/):
[https://minikube.sigs.k8s.io/docs/start/linux/](https://minikube.sigs.k8s.io/docs/start/linux/).

*I work on linux. Don't ask me about other operating systems*.

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
Here's the instructions to install kubectl cli:
https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux.
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
It's the main box. Usually I identify the cluster with the entire installation. It's your group of machines:
[https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-cluster](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-cluster).
The machines are the nodes.

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

"Pods are used as the unit of replication in Kubernetes." (_[Daniel Sanche](https://link.medium.com/QOXL6B9iC0)_).

A pod contains a **docker [container](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-container)**.

*I said "usually" because sometimes it could contains more than one container, and the container could be other than docker, [like rkt](https://containerjournal.com/topics/container-ecosystems/5-container-alternatives-to-docker/).
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

## Shut up and play
In the next step we'll play with some kubectl commands, stay tuned.

----
# Let's play with kubernetes
In the previous article we have configured minukube, the local kubernetes cluster.

Start it:
```
$ minikube start
minikube v1.5.2 on Linuxmint 19.2
Automatically selected the 'virtualbox' driver (alternates: [none])
Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
Preparing Kubernetes v1.16.2 on Docker '18.09.9' ...
Pulling images ...
Launching Kubernetes ...
Waiting for: apiserver
Done! kubectl is now configured to use "minikube"
```
Clone the example repo:
```
$ git clone git@github.com:batdevis/hello_kubernetes.git
```

We have some kubernetes configuration files:
```
$ ls *.yaml -1

deployment.app.yaml
deployment.nginx.patch.yaml
service.app.yaml
```
Let's create our kubernetes components into minikube cluster.

The deployment definition contains an example application that echo something:
https://hub.docker.com/r/hashicorp/http-echo.

```
$ kubectl apply -f deployment.app.yaml
deployment.apps/echotab created
```
The service, that maps the port from 80(external) to 5678(pod):
```
$ kubectl apply -f service.app.yaml
service/echotab created
```

And the ingress, that expose the service to the real cruel world.

The ingress component is the only kubernetes component that is always different according to your cloud provider.

The following commands works for minikube.
```
$ minikube addons enable ingress
$ kubectl apply -f ingress.echotab.yaml
ingress.networking.k8s.io/echotab-ingress configured
```

Copy and paste them and don't worry about how they works, for now.

When you'll setup a kubernetes cluster in a cloud provider, [check](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress) [out](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html) [their](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-on-digitalocean-kubernetes-using-helm) [guides](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/#additional-controllers).

Have a look to your components:
```
$ kubectl get all
```

And test your application:
```
$ curl -X GET $(minikube ip)
hello mister Tab
```
Oh oh oh we are live!

Let's check the status of the deployment and its pods.
```
$ kubectl get deployments
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
echotab   1/1     1            1           102m
```

```
$ kubectl describe deployments/echotab
Name:                   echotab
...
Selector:               app=echotab
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
...
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=echotab
  Containers:
   echotab:
    Image:      hashicorp/http-echo
...
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned default/echotab-5dd7fbb757-qglgc to minikube
...
```

```
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
echotab-5dd7fbb757-qglgc   1/1     Running   0          105m
```

```
$ kubectl describe pods/echotab-5dd7fbb757-qglgc
Name:         echotab-5dd7fbb757-qglgc
...
```

```
$ kubectl logs pods/echotab-5dd7fbb757-qglgc
2019/12/10 11:58:23 Server is listening on :5678
...
```
_The name of your pod will be different._

The commands above are the most used kubectl command, memorize them:

```
$ kubectl get pods/$POD_NAME
$ kubectl logs pods/$POD_NAME
$ kubectl describe pods/$POD_NAME
```

In case of pods in error status, these commands could save your day.

Delete the pod:
```
$ kubectl describe pods/echotab-5dd7fbb757-qglgc
```
And check the status in an other terminal:
```
$ kubectl get pods
NAME                       READY   STATUS        RESTARTS   AGE
echotab-5dd7fbb757-lzsnr   1/1     Running       0          10s
echotab-5dd7fbb757-qglgc   1/1     Terminating   0          112m
```
Look here, you can't kill the pod!

When a pod goeas down, the deployment turn up a new pod.

This is one of main concepts of kubernetes.

The deployment is responsible to keep the number of desired replicas of the pods up.

Try to increment the number of replicas in deployment.app.yaml:
change
```
...
spec:
  selector:
    matchLabels:
      app: echotab
  replicas: 1
...
```
in
```
...
spec:
  selector:
    matchLabels:
      app: echotab
  replicas: 2
...
```

Apply the modifications:
```
$ kubectl apply -f deployment.app.yaml
deployment.apps/echotab configured
```
Say hi to the new pod:
```
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
echotab-5dd7fbb757-87znq   1/1     Running   0          24s
echotab-5dd7fbb757-lzsnr   1/1     Running   0          12m
```

Ok, it's enough for today.

Clean the scene of crime:
```
$ minikube stop
Stopping "minikube" in virtualbox ...
"minikube" stopped.

$ minikube delete
Deleting "minikube" in virtualbox ...
The "minikube" cluster has been deleted.
Successfully deleted profile "minikube"
```

Bye!
