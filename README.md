# Kubernetes

## Contents

-   Need of Kubernetes
-   Kubernetes basic architecture
-   Kubernetes basic concepts (pods and containers)
-   Life cycle of a pod
-   Kubernetes Configuration
-   Kubernetes setup – Install Kubectl and Minikube
-   Deployment and StatefulSet
-   Namespaces
-   Interacting with pods
-   Configmap and secrets and volumes
-   Configmap in detail
-   Secrets in detail
-   Replica Set
-   Node affinity, Taints and Toleration
-   Node selector
-   Service Accounts in Kuberenetes
-   Security Context in Kuberetes

## Need of Kubernetes

Kubernetes is an open-source container orchestration tool developed by Google. It helps to manage containerized applications in different deployment environments like physical machines, virtual machines or cloud environments.

What is the need for an orchestration tool?

-   Trend from monolith to microservices.
-   Increased use of containers. Containers actually offers the perfect host for small independent applications like microservices. And the rise of containers make managing of 1000s of containers in multiple environments become tedious job.

What problems does Kubernetes solve? What are the tasks of an orchestration tool?

-   High availability or no downtime for applications (ie always available to users)
-   Scalability or high performance : means application has a high performance, it loads fast and the users have a very high response rates from the application
-   Disaster recovery ie backup and restore

## Kubernetes basic architecture

![image](https://user-images.githubusercontent.com/106816732/213409273-d2e40be3-0fc0-4ff1-a09d-f7261bee5584.png)

![image](https://user-images.githubusercontent.com/106816732/213409297-b1c21ecd-cbdf-4f71-af01-28eb8f83d7b5.png)

The Kubernetes cluster is made up with at least one master node and connected with few worker nodes. Each node has Kubelet process running on it. Kubelet is a Kubernetes process that makes it possible for the cluster to talk to each other and executes some tasks on those nodes like running application processes.

Each worker node has docker containers of different applications deployed on it. So depending on how the workload is distributed, there would be different no of docker containers running on worker nodes. And worker nodes are where the actual work is happening, so here is where your applications are running.

So the question is what is running on master node? Master node actually runs several Kubernetes processes that are absolutely necessary to run and manage the cluster properly.

![image](https://user-images.githubusercontent.com/106816732/213409337-e215b169-dc1f-4c73-b059-7c5ec82e0302.png)

One of such processes is an API server which also is a container. An API server is actually the entry point to the kubernetes cluster. So this is the process which the different Kubernetes clients will talk to. Like UI, if you're using kubernetes dashboard, an API, if you're using some scripts and automating technologies and a CLI. So all of these will talk to the API server.

Another process that is running on master node is a controller manager which basically keeps an overview of what's happening in the cluster whether something needs to be repaired or maybe if a container died and it needs to be restarted etc.

Another one is scheduler which is basically responsible for scheduling containers on different nodes based on the workload and the available server resources on each node so it's an intelligent process that decides on which worker node the next container should be scheduled on based on the available resources on those worker nodes and the load that container meets.

Another very important component of the whole cluster is actually an etcd key value storage, which basically holds at any time the current state of the kubernetes cluster. So it has all the configuration data inside and all the status data of each node and each container inside of that node. The backup and restore that we mentioned previously is actually made from these etcd snapshots because you can recover the whole cluster state using that etcd snapshot.

Kubelet – is a node agent which manages the whole node. Ie it tracks, stores, creates , updates or delete the containers within a node. Moreover it connects with the API server.

Kubeproxy – is sued for network proxy.

Another very important component of Kubernetes which enables those nodes, worker nodes, master nodes talk to each other is the virtual network, that spends all the nodes that are part of the cluster and in simple words virtual network actually turns all the nodes inside of the cluster into one powerful machine that has the sum of all the resources of individual nodes.

One thing to be noted here is that worker nodes actually have most load because they are running the applications inside it usually are much bigger and have more resources because they will be running hundreds of containers inside of them. Whereas master node will be running just a handful of master processes so it doesn't need that many resources however master node is much more important than the individual worker nodes because if for example you lose a master node excess you will not be able to access the cluster anymore and that means that you absolutely have to have a backup of your master at any time. So in production environments usually you would have at least two masters inside of your kubernetes cluster. But in more cases, you're going to have multiple masters where if one master node is down the cluster continues to function smoothly because you have other masters available.

## Kubernetes basic concepts (pods and containers)

![image](https://user-images.githubusercontent.com/106816732/213409382-538fc03c-1ea0-41f1-befb-9ac191b0d72c.png)

In Kubernetes pod is the smallest unit a kubernetes user will configure and interact with. Pod is basically a wrapper of a container and on each worker node there will be multiple pods and inside of a pod there might have multiple containers. Usually for one application there would have one pod. The only time multiple containers occur inside of a pod is when there is a main application that needs some helper containers. Example; a database would be one pod. a message broker will be another pod, a server will be again another pod. Node js application or a java application will be its own pod.

Virtual network inside kubernetes cluster assigns each pod its own IP address. So each pod is its own self containing server with its own IP address. And the way that they can communicate with each other is using the internal IP addresses. **We don't actually configure or create containers inside of Kubernetes cluster but we only work with the pods which is an abstraction layer over containers and pod is a component of kubernetes that manages the containers running inside itself without our intervention**. If a container stops or dies inside of a pod it will be automatically restarted inside of the pod. However pods are ephemeral components which means that pods can also die very frequently and when a pod dies a new one gets created and here is where the notion of ‘**service’** comes into play. So what happens is that whenever a pod gets restarted or recreated a new pod is created and it gets a new IP address. For example if our application talking to a database pod using the pod’s IP address and when the pod restarts it gets a new IP address which is very inconvenient. So because of that another component of Kubernetes called service is used which basically is an alternative or a substitute to those IP addresses so instead of having this dynamic IP addresses, there will be services sitting in front of each pod that talk to each other. So now if a pod behind the service dies and gets recreated and the service stays in place because their life cycles are not tied to each other and the service has **two main functionalities one is an IP address so it's a permanent IP address which you can use to communicate between the pods and at the same time it is a load balancer.**

## Life cycle of a pod

Whenever we launch a pod, it will go to the ‘pending’ stage or the creation stage where the image is being pulled from the hub. Once the image is pulled and all ready, status will enter into ‘running’ stage. If we initiate the deletion of pod command, the status will go to ‘deleting’. If the image is not present in the hub, the status will be ‘failure’. If its failure, we can set some kind of policies like ‘Restart Always’ where it initiates some ‘pull again’ policy

## Kubernetes Configuration

All the configurations in kubernetes cluster actually goes through a master node with the process called API server. So kubernetes clients which could be a UI, a Kubernetes dashboard for example or an API which could be a script or curl command or a command line tool like KubeCTL, they all talk to the API server and they send their configuration requests to the API server which is the main entry point or the only entry point into the cluster. The requests have to be either in YAML or JSON format. An example configuration in the YAML format actually looks like

![image](https://user-images.githubusercontent.com/106816732/213409431-815b6420-9c91-4260-b6de-850ad96a1bd2.png)

This is a request to Kubernetes to configure a component called deployment which is basically a template or a blueprint for creating pods and in this specific configuration example we tell kubernetes to create two replica pods called my app with each pod replica having a container based on my-image running inside. In addition to that we configure what the environment variables and the port configuration of this container inside the pod should be.

And as we can see the configuration requests in kubernetes are in declarative form. So we declare what is our desired outcome from kubernetes and Kubernetes tries to meet those requirements. For example since we declare we want two replica pods of my app deployment to be running in the cluster and if one of those pods dies the controller manager will automatically restarts the second replica of that pod.

## Kubernetes setup – Install Kubectl and Minikube

Lets setup a single node Kubernetes cluster having 1 master node and 1 or more worker node. There are several ways to setup this in system.

1.  Docker desktop (Kubernetes comes inside docker)
2.  Minikube (for more flexibility . Can customize the size, memory etc of cluster. Majorly used )
3.  Kind (fast, build using node js)
4.  Microk8s – multi node cluster
5.  K3S – helps in light weight distribution of Kubernetes. Build by rancher.

There are two ways to interact with Kubernetes cluster

1.  Kubectl
2.  Minikube

**1 – Docker desktop setup**

Go to docker desktop Settings Kubernetes Enable Kubernetes Apply & Restart

All the components in the Kubernetes architecture will be provisioned in sometime in Containers/Apps area. Bottom left side will show two green bars saying Kubernetes is running and docker engine is running.

Note that Kubectl will also be installed automatically. Commands to check whther Kubernetes cluster has successfully installed in system are

-   Kubectl
-   kubectl get nodes

![image](https://user-images.githubusercontent.com/106816732/213409504-896882c3-971c-45b6-b8cc-71cf692c85af.png)

**2- Minikube setup**

<https://minikube.sigs.k8s.io/docs/start/>

## Deployment and statefulSet

![image](https://user-images.githubusercontent.com/106816732/213409992-839e16be-b572-4e6c-9316-e5638ecd13cc.png)

Say we have our ML model or web app and we need to deploy into KB cluster. We can do that using KB deployment. We write deployment.yaml file, which contains all the configurations to deploy onto KB cluster.

We have two types of applications : stateless and stateful. Applications which wont store any kind of data are known as stateless and viceversa. So stateful applications requires some amount of storages like KB volumes. In default it is a stateless application, once we link some kind of volumes to it, it becomes stateful.

![image](https://user-images.githubusercontent.com/106816732/213410034-758c62bc-fe67-4530-9ecf-0978eb1ce516.png)

**Stateful and Stateless App**

Applications which require to store the current state of the application is called stateful apps and viceversa. Lets say we have a java application which collects the data inputted by a user in chrome end point and forwards it in the form of JSON format to a db (MySQL) to store. MySQL here is the stateful app and Java app is the stateless app. Because DB is used to store the state of your application.

## Namespaces

Kubernetes namespace is a technique to create different groups . by default there are 3 groups available; kube-public, default and kube-system. All the dependencies we require will be installed in the ‘default’ namespace unless or until we creates a new one. Default namespace is where every KB resources is located by default. Until new namespaces are created, the entire cluster resides in default. kube-public namespace is used for public resources where we can dump any of the packages. Kube-system namespace is meant for Kubernetes components, so we don’t need to touch here.

Since the above three namespaces are default, we can create our own customised namespaces , say for example, development and production inside which we can write the respective dependencies. Suppose we are working in a team where it has production and development environments. Both env might have different dependencies and configurations. Say for example, dev needs tools like Jenkins, argoci and helm. Prod needs circleci, Jenkins and helm. So we can create n no of namespaces and n number of applications inside it.

Important property of namespaces is that every namespaces are isolated. Whenever we access a pod inside the cluster we need to mention the namespace like kubectl get pod -ns \<namespace name\>

![](media/6ce44e4e0eb14a2b10ca34d704a76767.png)

Whenever we create a new pod, or any kind of object like deployment or services or config map and secrets, this will default goes to the ‘default’ namespace. The following commands shows namespaces and create a new namespace.

![image](https://user-images.githubusercontent.com/106816732/213410151-5f0cf57e-32ad-417f-8d74-1aa97a5b7ea0.png)

The following command will create an nginx pod by taking the nginx image from docker hub and put it into inside ritesh namespace. If we don’t give ‘-n ritesh’, the object will go to default namespace.

![image](https://user-images.githubusercontent.com/106816732/213410200-a5b4bf40-b26a-4d52-99f6-acd1a7d73612.png)

If we don’t give -n \<namespace name\>, command will get pods in default namespace.

To delete a namespace : kubectl delete ns ritesh

## Interacting with pods

**Kubectl is the CLI used for interacting with the Kubernetes cluster. We make use of this in order to interact with the pods as well. If we look into the architecture, inside a worker node , at the bottom layer will be having kubelet and kube proxy. On top there will be containerization tool which is docker. On top there will be pods. Inside pod we are having a container and inside the container our application is running in the form of image just like an image is running inside a container in docker.**

Inorder to interact with a pod we need a) Kubectl setup b) K8s cluster up and running

Commands :

-   kubectl run nginx –image=nginx:latest [create an nginx pod]
-   kubectl get pods
-   kubectl get pod -o wide [This will show details of worker nodes as well]
-   kubectl get pod \<pod name\> [look into single pod configuration]
-   kubectl describe pod \<pod name\> [in detail including namespace details, pod life cycle, volumes attached, env variables etc]
-   kubectl get pod \<pod name\> -o yaml \> filename.yaml [get the describe output in a yaml file]
-   kubectl run nginx1 –image=nginx –dry-run=client -o yaml \> filename.yaml [indicates don’t apply the actual configuration, first show the yaml file, then I will manually apply the configurations.]

![image](https://user-images.githubusercontent.com/106816732/213410258-1671c6dc-dc17-41d7-b5ac-146642dba098.png)

We can see no pod named nginx1 has created. Now open the yaml file created, make the necessary changes made.

![image](https://user-images.githubusercontent.com/106816732/213410286-b8d67718-3323-4d59-9d60-e31a59f91525.png)

-   kubectl apply -f \<filename.yaml\> [applying the configuration define dinside filename.yaml to KB cluster]

## Configmap and secrets and volumes

Configmaps, secrets and volumes are the ways to store information inside KB cluster.

![image](https://user-images.githubusercontent.com/106816732/213410405-ee09292e-e5ec-42cd-9589-55853f24d38a.png)

**![](media/e5775764b47a22302c3a44c99bbbd4f7.png)**

![image](https://user-images.githubusercontent.com/106816732/213410453-472db75c-ac0f-4305-b78c-ccb95d5f128b.png)

[**https://kubernetes.io/docs/concepts/storage/persistent-volumes/**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

### Configmap In detail

Suppose we have a KB cluster which contains a front end application, back end module which makes communication with some kind of database for eg SQL. We need to store DB username and password. How do we store these credentials inside the cluster? If our credentials data is less than 1 mb we have configmap, which will store the data in the format of key value pair.

Configmap is a separate KB object or a component. We will launch configmap similar to the pod and will mention credentials inside it. There are 3 methods to create configmap. Using --from-literal , --from-file and yaml file.

![image](https://user-images.githubusercontent.com/106816732/213410518-38644555-72ca-4872-939b-29f306d18e67.png)
Commands :

-   kubectl get configmap [lists all configmaps. Note that kube-root-ca is default file]
-   kubectl delete configmap \<configmap name\>
-   kubectl describe configmap \<name\>

    method 1

![image](https://user-images.githubusercontent.com/106816732/213410554-8f1b612e-9c8a-475e-94c9-c31b10e74a4f.png)

    Method 2

    Create files user.txt which contains username and pas.txt contains password. Now run the command,

    kubectl create configmap hello --from-file=user.txt --from-file=pass.txt

    Method 3 -YAML file (declarative approach. Above 2 are imperative approaches)

   ![image](https://user-images.githubusercontent.com/106816732/213410596-47033737-040b-493e-b67a-afdbb0a0560c.png)

-   kubectl apply -f filename.yaml
-   kubectl describe configmap \<name\>

### Secrets in detail

In configmaps we store data in the format of key value pair. The data should be \< 1 mb. We can attach this to a pod as it’s a different component in the Kubernetes. One disadvantage of configmap is the data is exposed. Secrets is the remedy for this. Secrets stores the information in key value pair itself. Value will be encrypted in the format of base64 string.

-   Kubectl create secret generic hello --from-literal=username=ritesh --from-literal=password=1234
-   Kubectl get secret
-   Kubectl describe secret hello
-   Kubestl get secret hello -o jsonpath=’{}’

![image](https://user-images.githubusercontent.com/106816732/213410647-47add740-01ce-4f6c-bfcc-09587092da8a.png)

We can see that data is encrypted and we can unfold it using the command shown in screenshot. We can also create secrets after --file-file

-   kubectl apply -f filename.yaml [applying the secrets to the cluster]

## ReplicaSet

One of the principle property of system design is HA (High availabile), ie our application should not face any downtime. Even if the users amount increases ie load increases, application should work perfectly. This property is achieved in KB using replica sets, which is an object inside KB. Using replica sets we can create different replicas of pod. If we set our desired state as 3, means 3 replicas of same pod will be available in cluster. In case if one pod goes down, another replica will automatically created.

How do we maintain the desired state of a pod?

The following code snippet is taken from Kubernetes documentation for ReplicaSet itself. Copy the code and create new yaml file in our project directory. Note the replica, container and image settings in the code.

![image](https://user-images.githubusercontent.com/106816732/213410785-c5a7c7a9-4c55-4778-951d-7e01fdcc9774.png)

Once the yaml is ready, apply the configuration using the command kubectl apply -f .\\replicaset.yaml Check the replica sets using kubectl get rs

Check whether 3 pods are created using the command , kubectl get pod

Even if we delete a pod using command kubectl delete pod \<pod name\>, the pod will be recreated.

## Node affinity, Taints and Toleration

Whenever we create a pod, the request will go through the Kube scheduler. Task of scheduler is to schedule this particular pod to a particular node. Node affinity is a property of pods that attracts them to a particular node or set of nodes.

This scheduling of pods takes place through resource request. First of all the kube scheduler look into a particular node and check whether the pod’s requested amount of resource is present in node. Resource can be memory, virtual CPU, disk space required or requested by pod. If the resource is satisfied, this pod will be schedule to this node.

**Taints and toleration** : Imagine a person ahs applied a mosquito repellent cream of brand 1. Say mosq 1 is intolerant to this cream and cant sit over this person because of this taint over his body. Say moq 2 has a tolerance for this brand1 and can sit over his body. Similarly we have several pods and a node for example. Node is having a taint of blue. So whatever pod has a tolerance = blue is able to schedule to this node. Ie when taint and tolerance is same then only that particular pod will schedule to a particular node.

What if we want to forcefully schedule a pod with specific configuration to node 1, not 2 or 3? We can create taint for this node 1 and toleration for this pod and we can match them and we can easily schedule them. **This is the main intention of using taints and tolerations ie to assign a particular pod to a node.**

![image](https://user-images.githubusercontent.com/106816732/213410868-c51759c0-39f7-4221-8f0d-282ee9165e35.png)

**Practical**

![image](https://user-images.githubusercontent.com/106816732/213410905-8a62ae27-3ae6-43bf-84ba-c60c62d97fc6.png)

Now node has tainted. Go to VS code for creating tolerance for pod.

![image](https://user-images.githubusercontent.com/106816732/213410935-32f6e1a3-74c7-4f63-92c5-2a9eb402f6f0.png)

Now apply the yaml. And check details using kubectl describe \<pod name\>

The default value for operator is Equal. A toleration "matches" a taint if the keys are the same and the effects are the same, and:

-   the operator is Exists (in which case no value should be specified), or
-   the operator is Equal and the values are equal.

<https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/>

## Node Selector

Say we have 3 pods and 3 nodes in our cluster. Usually the pods are getting assigned to the nodes are at random. Suppose 1 node 1 is very heavy ie it has very high configuration. Other 2 nodes are small. And suppose pod 2 requires high memory since it has heavy computational tasks inside. So if pod 1 is added to node 1 and pod 2 into node 2/3, its illogical.

So whenever we create a pod we add new label inside it named as node selector. We can label a particular node as key : Large. Ie we are labelling a node that its large. So whenever a pod gets created and if it needs heavy computation it will get assigned to node 1 by using a tag known as node selector.

How to create a node selector?

Label a particular node : kubectl label nodes \<node-name\> \<label-key\>=\<label-value\>

![image](https://user-images.githubusercontent.com/106816732/213411021-0f7004d4-5229-4223-b99a-002622bb3615.png)

In the above screenshot, there is only one node which is docker-desktop. Then we label that node by giving a key value pair hello=world.

Once the node is labelled, we can create a pod specification yaml file, inside which we can pass the label known as node selector. Go to VS code and create a new folder for node selector under the project and create the yaml file. The code here is copied from KB documentation and edited for our requirements. Note the nodeselector option and taint option.

![image](https://user-images.githubusercontent.com/106816732/213411093-e2193a30-954c-436b-a245-769ffa528299.png)

Once the yaml is ready apply it.

**CronJob**

Cron jobs are for schedule a particular action. Lets say we need to show hello world every 5 mins, or take backup of data from KB cluster every hour, we can create cron expressions. Refer Documentation.

## Service Accounts in Kubernetes

In KB we have user account and service account. The user account is mainly for granting permissions for users. Ie if I want to attach some kind of RBAC (Role based authentication Control) in KB cluster I will create user acc and attach some kind of security policies depending upon roles. So whenever we want to grant some kind of authentication or authorization to a particular resource in KB we use user acc.

Suppose I have Jenkins to deploy something into KB and Prometheus tool for scrapping logs. For that I have to provide some kind of authentications to these tools to access KB. This is done through service accounts. So the main use of svc acc is to provide some kind of access to a particular service.

Whenever a pod get created, a default service account will be created.

![image](https://user-images.githubusercontent.com/106816732/213411136-1c2fe8ae-4d3e-49bd-9bfa-6ff315ab91b4.png)

Note the token of service account. It is encrypted using a secret. We can see this token in secrets as well using the command ‘kubectl get secrets’ . We can use this token as a bearer token to make a particular authentication between the services. Lets say id I have attach this particular token inside Jenkin, so that I can easily publish the objects to the KB cluster or easily deploy some objects using Jenkins automation server.

![Uploading image.png…]()

The above screenshot shows the token. Attach this token in a particular curl request. Lets say I want to create a pod using Jenkins. Attach the above token to the Jenkins dashboard, then I can easily perform a shell command of kubectl run pod.

## Security Context in Kuberetes

KB architecture says, our application or ML model will run inside a container and container resides inside a pod. Now we need to set the security principles for the cluster. Either we can give pod level security congig or container level. Suppose we write security configuration at both pod level and container level. **The container level security config will overrides the pod level.** We can distribute pod level security config into three parts. Run as user (Personal level privilege to access files, codes etc of a pod) , run as group (group level privilege), fsgroup (these policies generally attach to the KB volumes).

Reference : <https://kubernetes.io/docs/tasks/configure-pod-container/security-context/>
