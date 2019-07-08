As overviewed in the [official document](https://kubernetes.io/docs/concepts/) 

> To work with Kubernetes, you use Kubernetes API objects to describe your cluster’s desired state: what applications or other workloads you want to run, what container images they use, the number of replicas, what network and disk resources you want to make available, and more. You set your desired state by creating objects using the Kubernetes API, typically via the command-line interface, kubectl. You can also use the Kubernetes API directly to interact with the cluster and set or modify your desired state.   
   
> Once you’ve set your desired state, the Kubernetes Control Plane makes the cluster’s current state match the desired state via the Pod Lifecycle Event Generator (PLEG). To do so, Kubernetes performs a variety of tasks automatically–such as starting or restarting containers, scaling the number of replicas of a given application, and more. The Kubernetes Control Plane consists of a collection of processes running on your cluster:



#### Understanding of very basic Kubernets Objects
 
> 1. [PODS](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)    
> ```
> kubectl get pods
> ```
> 2. [SERVICES](https://kubernetes.io/docs/concepts/services-networking/service/)    
> ```
> kubectl get services 
> ```
> 3. [NAMESPACES](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)    
> ```
> kubectl get namespaces 
> # In QCC you are restricted to one namespace
> ```
> 4. [REPLICA SETS](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)  
> ```
> kubectl get rs
> ```
> 5. [DEPLOYMENTS](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)   
> ```
> kubectl get deployments
> ```


