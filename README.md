# Automate kubernetes deployment with ansible

DIY guide for Learning kubernetes (k8s):

- How to install and remove a k8s cluster
- basic fundamentals
- Learning by automating all the installation and configuration process with ansible.

I encourage to do it yourself: You should try to install a k8s cluster with 
containerd, kubeadm and calico using an external tutorial, 
then try to automate the process
and use the sources present on this repository as a back up.

If you're able to improve my work, please feel free to make a pull request suggesting the new changes.
Thank you!

# K8s fundamentals

- kubernetes (k8s) is a **Container Orchestrator** at its core, but it's highly extensible
and customizable. Hence, some people consider k8s a full-fledged infrastructure
platform.
- It's a **workload Placement**: We can specify where and how a container is deployed (replicas, running on specific server with special resources...)
- It provides **Infrastructure abstraction**: k8s can perform a lot of things for us, like load balancing loads, create network configuration... Thus, k8s provides a layer of abstraction.
- It's the job of k8s to maintain the system in a **desired state**. That state can be created declaratively using code.

## Principles

1. We write deployment code to specify the ```desired state``` of the cluster. K8s will be responsible for read that code and perform the magic.
2. The magic is performed thanks to ```controllers``` (also called controller loops)
that are constantly monitoring the system to make sure that the it is in that
desired state, making any modification if required.
3. And the magic can occur because of the ```k8s API``` (also know as The API
Server) which is the glue between everything, we deploy our desired state using
the API. The different parts of the system communicates with each other 
using this API as well. **Is the centralized Hub used for communication**.

> k8s API defines a set of object or resources, that are the information units used for
representing the system state.

## API Objects

API Objects are a collection of primitives to represent the system state. e.g: pods, nodes...

> [!TIP]
> API objects are represents the state of our system.

> [!TIP]
> You can get all the k8s cluster resources with the command
> kubectl get all --all-namespaces

Thanks to this objects we cant configure the state either declaratively (creating a file where we will declare the different
objects and deploy them afterwards) or imperatively (using cli commands
directly).

> [!NOTE]
> The files where we specify declaratively our API Objects (our desired state)
> are called Manifests

> The API server has a RESTful API that runs over HTTP using JSON, and it's what
we use as administrators to interact with k8s.

> [!NOTE]
> We don't normally interact with the API ourselves directly. We generally use 
> CLI or GUI tools for interacting with the cluster. These tools do the ugly work
> of dealing with the API, and in return they offer a more user-frienly
> environment to us.

> [!IMPORTANT]
> API objects are organized by `kind` (Pod, Service, Deployment...), `Group`(core, apps, storage) and `Version` (v1, beta, alpha)

### Pods

They are a single or a collection of containers **deployed as a single unit**.

- Can be composed for one or more containers
- Basic unit of work
- Unit of scheduling 
- Ephemeral - no pod can be 'redeployed', the are either created or destroyed.
- Atomicity - They are in the cluster or not. If only one container in the pod dies, the pod will be unavailable.

> [!TIP] How can k8s knows if a container if OK? Well, easy, monitoring the pod state... 
> But what if the application running in my pod requires a more complex health 
> check, How can I make k8s aware of that custom situation? With 'probes'. We can 
> configure probes for our pods to check their health.

> [!NOTE]
> By single unit we meant that if we create a pod with multiple containers, all the 
containers will be managed together: creation, deletion...

### Controllers

Keeps the system in the desired state

- They monitor pods state and health
- They make the required modifications to the system for achieving the desired
state
- Therefore, they create and manage pods
- They respond to the pod **state** and **health**

#### ReplicaSet

Is a controller object that is capable of manage pods. e.g.: If we
create a replicaset for a pod where we specify that we require 3 replicas
of the pod in different nodes of the cluster, is the replicaset's job to
creates, removes and distribute the pod in order to met the desired state.

> [!WARNING]
> We don't usually create replicaset objects
> directly. We use an intermediate controller object that offer a higher layer
> of abstraction called `Deployment`.

#### Deployment

Is a controller object used for creating replicaset objects, they monitor the state of the replicaset. Thus, with this higher abstract object we can, for example, roll out a replicaset that run an specific version of a container.

### Services

Provides a **persistent access** to the containers on our pods.

If we have a pod with a container where an application is running, and that pod is replicated in different nodes of the cluster. How can we access that application? Should we specify the node where the application run? What happen if a pod in a node crashes?. Services are responsible for offering an endpoint for the application and deal with all those issuesand more.

- Adds persistency to the ephemeral world of pods.
- It's the networking abstraction for Pod access.
- It provides a persistent IP and DNS name for the service.
- No matter what happened to the pods (if they are updated, deleted or created),
they will make the necessary networking plumbing for connecting everything.
- It provides options for configure scalability.
- It provides load balancing.

### Storage

Objects required when our containers requires a persistent storage.

### Organizing objects in k8s

#### Namespaces

- Subdivide a cluster and its resources
- Conceptually a "virtual cluster"
- Deploy objects into a Namespace
- Resource isolation/organization

They are a security boundary for RBAC (Role-based Access control): We can limit WHO can access WHAT based on namespaces

You can have a resource called X deployed in several namespaces, no conflict name will arise. 

> [!IMPORTANT]
> Not all objects can be namespaced: Pods, Controllers, services can whereas "Physical things of the cluster" can't PersistentVolumes, Nodes

> [!TIP]
> kubectl api-resources --namespaced=false
> will list all api resources that can't be namespaced

##### Default namespaces in a k8s cluster

- **default**: When you deploy resources and you don't specify a namespace, the resources will be deployed in this namespace
- **kube-public**: Is readable for all users (even those not authenticated) and it's commonly used for shared object between namespaces, for objects like `config-maps`
- **kube-system**: The system objects: pods like api-server, etcd, controllers, proxy, scheduler....


#### Labels

To group certain objects and act on them as a whole.

> [!IMPORTANT]
> Some resources will need
> a 'selector' like a label in order to know on which 
> resource they should act on.

E.g: We can group with a label a bunch of `pods` and make then
accessible using a `service` which is a resource that will need
a selector.


#### Annotations

For adding a little more of metadata about a particular object or resource

> [!IMPORTANT]
> They are similar to labels (key/value pairs) but they can't be used
> to query other resources...

## k8s Architecture Overview

![01](.doc_assets/k8s_architecture.svg)

### Control plane node

> Previously called Master node

It coordinates cluster operations, monitoring, pod scheduling and is the primary access point for cluster administration.

It's comprised of several components:

#### API server

- It's the primary access point for the cluster and administrative operations. Thus, **it's the communication HUB for k8s cluster**.
- it provides as RESTful API over HTTP with JSON for interacting with k8s.
- This element is ```stateless```, which means that it doesn't store any information about the system.

Normally, instead of communicating with k8s using the RESTful API, we can use
command line interfaces (cli) like ```kubectl``` (there are many others). To interact with the k8s cluster.

> [!WARNING]
> kubectl **IT'S NOT PART OF THE CONTROL PLANE**, but it's worth mentioning. Is a tool 
> created for making your life easier, you type commands with kubectl and in turn he translate them
> to API calls to the API server.

#### etcd

- Created for providing the API server component with storage.
- Responsible for persisting the state of k8s objects in the cluster.
- Is updated only by the API server
- It stores the API objects using key-value pairs

#### scheduler

- Tells k8s which nodes to start pods on, monitoring the API server for unscheduled pods.
- For making the scheduling possible, it will evaluate pod resources in terms of things like CPU
memory, storage requirements to ensure their availability when placing a Pod
on a specific node in the k8s cluster.
- It respect any restriction or constraint defined administratively, things like keeping two pods on the same node at all times (pod affinity) or the opposite (pod anti-affinity).

#### Controller manager

- Has the job of implementing the life cycle functions of the controllers that
execute and monitor the state of the k8s objects such us pods. 
- They keep things in the desired state.
- They watch the API server and update it if required to ensure that its heading towards the desired state.

#### Kubelet

We will describe this component in the next section in further details.

Kubelet is a system daemon or service, this service deals with the container runtime.
Thus, kubelet and a container runtime is required and are fundamental pieces in 
ALL the k8s cluster nodes.


Now, I want to clarify something:

I saw many other diagrams on the internet that didn't include kubelet
in the control plane node. So you might think that kubelet is only required
in worker nodes, but that's absolutely wrong...

Why this component is depicted only in worker nodes by the community?

I guess that they
don't want you to make the expectation *that pods can run in control plane nodes*.
Statement that it's true by default:
Control plain nodes are '**tainted**' and configured to only run system containers not users pods.

But, we must know that that's only configuration that can be changed if we want to (Something that's is not recommended
for production environments).

> [!WARNING]
> Many of the k8s components like etcd, the controller manager, the API server are CONTAINERS
> that run on a container runtime. Thus, kubelet is required in every single node,
> otherwise, the control plane node can't spin those container so please DO NOT CONFUSE THEM WITH OS SERVICES.
> The only OS service in a k8s cluster is kubelet.

> [!NOTE]
> In this guide the utility kubeadm
> will install it as a systemd service in Linux

#### Container runtime (CRI)

Similarly, this component will be explained in the next section, and is also required in all
the nodes.

### Node or Worker node

- Nodes where the pods run, their responsibility is to run pods. Hence, services
- They also implement network reachability to the pods and services
- They can be either, virtual or physical machines

In a nutshell each worker node in a cluster contributes to the compute capacity of the cluster.

#### Kubelet

- It's responsible for starting containers (hence, pods) in the node, 
dealing with the container runtime
- It communicates directly with the API server in the control plane node
- Monitors API server for changes
- Responsible for pod life cycle
- Reports Node & Pod state
- It's responsible for running the pod probes

#### kube-proxy

- Responsible for pod networking (iptables) and routing traffic to pods
- Load balancing is implemented in this component
- Responsible for implementing our services abstraction on the node itself
- It communicates directly with the API server in the control plane node

#### Container runtime

Actual runtime environment for the pods containers

- They pull container images (download) from a container registry
- They provide the execution environment (run containers)
- It's required for a container runtime to implement the container runtime interface
(CRI). This interface is what k8s uses for managing the containers. Therefore,
we can change this element for any piece of software that implements the CRI.
A typical CRI could be containerd.

> [!NOTE]
> In this guide we will install containerd.

#### Special pods in k8s

Special-purpose Pods, are pods that might be optional but they are
extensively common in a k8s cluster around the world. Normally, those pods are
deployed by other projects like ArgoCD, Prometheus, KEDA....

> [!TIP] 
> This is important because we must be aware of this special pods. They will shape the k8s
cluster behaviour and extend their functionality.
> One k8s cluster can be completely different from another because of these 'special' pods. You can check this [link](https://www.cncf.io/projects/) to
> have a glance of the extremely variety of them.

> [!NOTE]
> In this guide we will use [calico](https://github.com/projectcalico/calico) for making possible
> the network connectivity between containers in the cluster.

##### DNS

Provide DNS service inside the cluster, commonly used for **service discovery**
for applications deployed in the cluster. You'll normally find DNS pods deployed
in nearly every k8s cluster.

##### Ingress controllers

They are advanced HTTP or layer 7 load balancers and content routers.

##### Dashboard

Pods that provides and interface for web-based administration of a k8s cluster 

## Pod Operations

1. Using kubectl we **submit code** to instruct k8s to create a **deployment**. In that deployment we've defined that we want 3 replicas of our pod.
2. That request is going to be submitted to the **API server**.
3. The API server **stores the information** of the objects in **etcd**.
4. The **controller manager**, which is monitoring the API server, discover that
the state has changed and it's responsible for **spin up** those 3 requested replicas creating
a new CONTROLLER resource like a **replica set**.
5. The scheduler is also monitoring the API server, when the controller resource is created, he will notice that the replicas are not yet scheduled. Therefore, it will tell the API Server that the pods need to be scheduled on nodes selected by him.
6. API server persist this scheduling information in etcd
7. On the other hand, the kubelets on the nodes are continuously asking the API Server for work and they are reporting their state.
8. API server based on the scheduler decision will spin up the pods requested in the replica set on certain nodes in the cluster.
9. When the pods are created and running the controller manager is monitoring the state of the replicas, thanks to the nodes which are reporting their state to the API server. If a node goes down and stop reporting, the controller manager will detect that the system is not in the desired state and will follow the previous steps submitting a new request for pods...

So as you can see here, the API server is the central unit, everything goes through it.

> Why the Control plane node doesn't handle any work? Good question, it might seems like we're wasting resources, but overloading a worker node is something that can happened with some frequency in a k8s cluster, some overloading
scenarios end up with the node going down, overwhelmed. Therefore, if we
configure control plane nodes to be worker nodes as well, we increase the
possibility of the control plane node to be overloaded which could lead to the node going down. As you can guess
if the control plane node goes down, everything stop working and the k8s cluster become inoperable. That's why it's not recommended, for production environments, to use control plane nodes as worker nodes too.

> [!NOTE]
> There are other security implications that are out of the scope of this guide. So please, in a prod
> environment do not configure the control plane node to run worker load.

## Service operations

In the previous section we focus on what happen in the cluster from a pod perspective, in this section we're going to focus on what happen on the outside of the cluster. Based on the previous example of deploying 3 pods in a replica set, we can also expose access to that pods with a service.

Imagine we configure a service running on TCP port 80 linked to that replica set

1. User from outside of the cluster will try to connect to the pods using the 
fixed and persistent service endpoint, so the will try to connect to the TCP port 80.
2. As the request from the user comes in, into the cluster, they will be load balanced to the all pods linked to the service.
3. Because we combined this example with a replicaset, if a pod is unavailable for whatever the reason. Is the responsibility of the replicaset controller to maintain the desired state. Therefore, he will remove the pod and deploy a new one
4. The cool part is that users accessing from the outside don't know anything about this happening. Behind the scenes, the service will stop load balancing requests to the failed deleted pod and will continue load balancing request across
the remaining pods.

## Networking fundamentals

- Every Pod gets its own IP address: There should be no need to create links between Pods and no need to map container ports to host ports.
- NAT is not required: Pods on a node should be able to communicate with all Pods on all nodes without NAT.
- Agents get all-access passes: Agents on a node (system daemons, Kubelet) can communicate with all the Pods in that node.
- Shared namespaces: Containers within a Pod share a network namespace (IP and MAC address), so they can communicate with each other using the loopback address.

<!--
Some scenarios

### Inside a pod (Container-to-Container)

You deploy only one pod that have multiple containers. That containers communicate with each other. Hence, the communication occurs inside the pod.

In this case, the communication will be over localhost using namespaces

### Pod to pod within a node

In this case one pod in node A wants to communicate with another pod in node A.

Pods aren't self contained like in the previous example, so they can't communicate 
over localhost. They will communicate over a layer 2 software bridge installed on the node.
-->

Further reading [here](https://opensource.com/article/22/6/kubernetes-networking-fundamentals)

## Useful commands with kubectl

Some links that might help:

- [kubectl cheat sheet](https://k8s-docs.netlify.app/en/docs/reference/kubectl/cheatsheet/)

- `kubectl config get-contexts`: Get all the k8s cluster you've configured in your client
- `kubectl config use-context <name>`: Select that k8s cluster with `default` namespace. All commands from that moment will run on this chosen cluster.
- `kubectl cluster-info`: Get information about the selected cluster
- `kubectl api-resources`: Very useful for writing manifest and understanding all the API objects, you should combine it with `kubectl explain <api-resource-name>`

### Pro tips

#### the --dry-run=client flag

This flag is extremely useful:

- But you can combine it with many commands to create API object files populated correctly:
    - kubectl create deployment ...  --dry-run=client -o yaml > file.yaml
    - kubectl run pod --dry-run=client -o yaml > file.yaml
    - ...

# Installation and Configuration

In this section we will deploy a full k8s cluster, I did it with virtual machines
but if you have the hardware you can do it on real nodes.

As this project is for learning purposes the vault pass is ```1234```

#### The -v6 or -v7 flag

This flag add a lot of verbosity to your kubectl commands. e.g:

```bash
kubectl get po -v6
```

#### The --watch flag is great for monitor a resource

## About Manifests

Manifest are files that can be written in either JSON or YAML. Their use if for declaring
and API object.

> [!IMPORTANT]
> At least the following properties MUST appear on a Manifest: `apiVersion`, `Kind`, `metadata` and `spec`

We can use the command:

```bash
kubectl apply -f <manifes_file>.[yml|json]
```

To tell the API server we want to apply the manifest changes to the system.

Don't forget about `kubectl create`, and `kubectl diff` and the usage of flag `--dry-run=server` or `--dry-run=client` to generate and validate your objects.

## Prerequisites

If you want to deploy the cluster using my automation the following requisites are mandatory:

- 2-4 machines shipped with Ubuntu 22.04, cable to communicate with each other
    - The same username and password are configured for SSH into all the nodes (Yes, I know this is not a good practice, but we're in a lab trying to learn k8s fundamentals...)
    - SSH is configured to accept authentication using username and password
- Machines should be accessible from the host that is running the ansible playbook. Alternatively, 
you can configure a bastion set up, which consist on configure only one machine to be accessible
from the host that is running ansible, and we use that machine as proxy to connect to the rest of
them.
- Python 3 and pip with venv installed on the host machine where ansible will run.

## Operations

> [!NOTE]
> You will need to modify the ansible inventory (```inventory.yaml```) to adjust to your machine
hostnames.

### On the host, create a python 3 virtual environment and install required dependencies

```bash
# vars, change these values according to your computer
$venv_path='./venv'
$src_path='./src' 

#Create a venv
python3 -m venv $venv_path

# source the virtual environment
source $venv_path/bin/activate

# install ansible and all python requirements
pip install -r $src_path/requirements.txt
```

### Configure ansible inventory according to your nodes

Change the file ```inventory.yaml``` according to your needs.

### Change ansible variables

According to your needs modify any variable in the files 

- ```group_vars/proxied_servers/vars```
- ```group_vars/all/vault```: Will require the use of the command ```ansible vaults```
- ```group_vars/all/vars```
- ```roles/kubernetes/defaults/main.yml```

### Test node connectivity from localhost

If you configured the bastion, you'll need to first login into each node of the cluster:

```bash
# ssh bastion 
# https://goteleport.com/blog/ssh-bastion-host/
ssh -C -o ControlMaster=auto -o 'User="k"' -o 'ProxyCommand=ssh -W %h:%p -q <proxy_user>@proxy_host' <target_host>
```

Finally check network connection with the cluster:

```bash
ansible-playbook -i inventory.yaml ping.yaml --ask-vault-pass
```

#### Deploy kubernetes cluster

```bash
ansible-playbook -i inventory.yaml create.yaml --ask-vault-pass
```

#### Destroy kubernetes cluster

```bash
ansible-playbook -i inventory.yaml destroy.yaml --ask-vault-pass
```

### Other useful commands

```bash
# To create a requirements.txt based on the packages installed in the current
# environment
pip freeze > requirements.txt

# To play with ansible vaults
ansible-vault ...

# creating ansible role
ansible-galaxy init my-role
```

# Troubleshooting k8s

## Debug node containers with nerdctl (containerd)

To list all the containers running in a node

```bash
sudo nerdctl -n k8s.io ps
```

> [!WARNING]
> You will need to install this utility

# Useful resources

- [Ansible roles](https://www.youtube.com/watch?v=SvcOwBFLVLM)
- [Ansible vaults](https://www.youtube.com/watch?v=MnBV8zLq-_Y)
- [how to start k8s cluster](https://www.howtogeek.com/devops/how-to-start-a-kubernetes-cluster-from-scratch-with-kubeadm-and-kubectl/)

# TODO

Some nice to have features in the guide and automation

- [ ] Automate vm creation on selfhosted target with packer + virtualbox
- [ ] Add compatibility for arm64 based machines
- [ ] Configure more than one control plane node
- [ ] Further explanation for the networking
- [ ] Enhance troubleshooting section
- [ ] Tutorial for exposing services outside the cluster
