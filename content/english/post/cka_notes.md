+++
author = "M0D4S"
title = "CKA Exam Notes: Core Concepts"
date = "2023-11-25"
description = "My Notes for preparing for CKA Exam"
tags = ["cka", "kubernetes", "k8s", "sysadmin", "notes", "cheatsheet", "cheat sheet", "cheatsheets", "cheat sheets", "cka notes", "cka cheatsheet", "cka cheat sheet", "cka cheatsheets", "cka cheat sheets"]
+++

# Core Concepts

# I. Cluster Architecture

## I.1. Master

- Manage
- Plan
- Schedule
- Monitor

***Components:***

- ***ETCd:*** Key-Value Store
- ***Kube-Scheduler***
- ***Kube-Controller-Manager***
- ***Kube-apiserver***
- **etc ..**

## I.2. Workers

- Hosts apps as containers.

***Components:***

- ***Kubelet***
- ***Kube-Proxy***
- ***Container Runtime Engine***

# II. CRE & Docker

- ***OCI:*** Open Container Initiative
    - Standardize Image Specs and Runtime Specs
- ***DockerShim:*** A way to keep docker compatible with OCI (Consequently K8S), but it was removed in Docker V1.24 as docker became OCI compatible starting with that version.
- ***Docker:*** was a set of tools (CLI, API, Build, Volumes, RUNC, containerd…)
- ***Containerd:*** is a daemon to manage RUNC (CRE), then was cut off of docker project and became standalone project. But, it is not production level.
- There is also:
    - ***NERDCTL:*** A whole CRE solution with its utilities.
    - ***CRICTL:*** Used to inspect and debug containers installed separately.
        - Order of connections to runtime endpoints:
            1. unix:///run/containerd/containerd.sock
            2. unix:///run/crio/crio.sock
            3. unix:///var/run/cri-dockerd.sock
        - Have to be set manually:
            
            ```jsx
            crictl --runtime-endpoint
            export CONTAINER_RUNTIME_ENDPOINT
            ```
            

# III. ETCd

- Port 2379/tcp.
- Key-Value Store
- Distributed
- Reliable
- Simple, Secure & Fast

```jsx
etcdctl set key1 value1
etcdctl get key1
```

- A lot of differences between VERSION2 & 3. Default is V2, but if V3 is needed,

 

```yaml
# Method1
ETCDCTL_API=3 ./etcd ...
# Method2
export ETCDCTL_API=3
./etcd ...
```

- ETCd is the inventory of the K8S cluster.
- ETCd only changes when the change is actually done.
- If ETCd is set up with kubeadm, it will run as a pod in the K8S namespace.
- In the unit file of the etcd service: `--advertise-client-urls https://${**IP**}:2379`

# IV. Kube-apiserver

- To contact it, either use the **kubectl** or just **send a request** to the API endpoint.
- If set up with kubeadm, it will run as a pod in the K8S namespace.
- It is the primary management component in K8S.
- Example of: `kubectl get nodes`
    1. Auth request
    2. Validate Request
    3. Retrieve/perform action
    4. Update ETCd
    5. 
    ```yaml
     Scheduler → API Server → ETCd
                                              → Kubelet → CRE
                                                                → API Server → ETCd
    ``` 

# V. Kube Controller Manager

- Purpose:
    - Watch Status
    - Remediate Situation
- Process
    - Continuous Monitoring
    - Bring the Cluster to the Desired Functioning State
- There is a lot of Controllers, for instance

## V.1. Node Controller

- Node Monitoring Period (5s)
- Grace Period (40s)
- Eviction Timeout (5m)
    - If a node is evicted from the cluster (5 Minutes since its last heart beat), There will be reprovisionning of the pods that were there (**If they belong to a ReplicaSet**).

## V.2. Replication Controller

- Deprecated (being replaced by ReplicaSet).
- Ensures that the desired number of replicas of a pod exists.

# VI. Kube Scheduler

- Decides which pod goes to which node.
- 2 Phases to decide:
    - Filter unsuitable nodes.
    - Rand the rest: Assign priority function (score on 10)

# VII. Kubelet

- Register the node with the K8S Cluster.
- It is manually installed on the workers.

# VIII. Kube Proxy

- Process that runs on each node in the cluster that creates rules to forward traffic sent to services to the actual pods.
- Pod Networking Solution.
- A service is a virtual component that lives in the K8S memory.

# IX. PODs

- Smallest entity that can be created by K8S.
- POD ↔ Container: One to One relationship
- ***BUT:***
    - Helper Containers can be in the same pod.
    - They refer to each other by localhost
    - Can easily share storage
    - **RARE CASE**

# X. YAML in K8S:

- Main properties:

```yaml
apiVersion: v1
kind: Pod # [ReplicaSet | Deployment | etc ...]
metadata:
spec:	
```

- ***Commands:***
    - `kubectl create -f pod_definition.yml`
    - `kubectl apply -f pod_definition.yml`
    - `kubectl describe pod [POD-NAME]`
    

# XI. ReplicaSet

- Replicates Pods
- Ensure X number of Pods are running.
    - It is an update on Replication Controller.
- `apiVersion: apps/v1` for ReplicaSet
- `apiVersion: v1` for Replication Controller
- For ReplicaSet, there is another mandatory fields under spec:
    - **template:** It contains the definition of template pods
    - **replicas:** the number of replicas
    - **selector:** (MatchLabels:) it matches labels to add already running pods to the ReplicaSet.
- Scaling a ReplicaSet can be done by:
    - Changing the yaml file and re-applying it.
    - `kubectl scale --replicas=25 -f file.yml`
    - `kubectl scale --replicas=25 replicatset [RS-NAME]`
    
    ⇒ Both commands only change the loaded conf, if the cluster restarts it will take the hard coded replicas number again.
    

  

# XII. Deployment

- Enable Rolling-Updates
- Undo changes
- Pause
- Resule changes

⇒ For pods.

- apiVersion: app/v1 (same as RS)
- `kubectl get all`

# XIII. Services

- Enables communications within & outside of the app.
- Use Cases:
    - ***NodePort:*** Forward the port to a host’s port.
    
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: kubia-nodeport
    spec:
      type: NodePort
      ports:
      - port: 80            # Mandatory
        targetPort: 8080    
        nodePort: 30123     # From 30000 to 32767
      selector:
        app: kubia
    ```
    
    - ***ClusterIP:*** Creates a Virtual IP inside the Cluster to enable communication between objects.
    - ***LoadBalancer***
- ***Commands:***
    - `kubectl get svc`
    - `kubectl describe svc [NAME]`
    

 

# XIV. Namespaces

- Default namespace is “default”.
- Namespaces created by default: default, kube-system, kube-public

⇒ They contain isolated policies.

- Quotas can be defined for each namespace.
- When calling a service outside the default namespace:

[Service-Name].[Namespace].svc.[DomainName_Of_K8S_Cluster]

- ***Commands:***
    - `kubectl get pods --namespace=kube-system`
    - `kubectl create -f pod-definition.yml --namespace=ProjectX`
        
        ⇒ Can also add `namespace: ProjectX` uner te metadata section.
        
    - `kubectl config set-context $(kubectl config current-context) --namespace=ProjectX`
    - `kubectl get pods --all-namespaces`
    - Can Also create NS using a yaml file:
    
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
    	name: ProjectX
    ```
    
    - ***Resource Quota:***
    
    ```yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: ProjectX-Quota
    	namespace: ProjectX
    spec:
    	hard:
    		pods: "10"
    		requests.cpu: "4"
    		requests.memory: 5Gi
    		limits.cpu: "10"
    		limits.memory: 10Gi
    	
    ```
    

# XV. kubectl APPLY

- Create the object from a yaml file if not created.
- K8S takes the yaml file and adds to it a status field & uses it as live conf (kubectl edit .. : will change the live version of the object)
- Every time a new conf applied to that object, it is transformed into json & saved.
- A comprarison between those three files happens each time a new update of the object is applied. That comparison enables K8S to decide what will it do.
    
    ***PS: DO NOT USE IMPERATIVE AND DECLARATIVE APPROACHES TOGETHER.***