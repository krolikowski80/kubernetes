# Requesting and Limiting Resources for Pods

#### Introduction
It is a best practice to include some information about the compute resource requirements for a Pod in its specification. The compute resources that Kubernetes includes built-in support for are CPU, measured in cores, and memory, measured in bytes. Kubernetes allows you to include a requested amount (minimum) and a limit (maximum) for each compute resource. Although the compute resource requests and limits are optional, the Kubernetes scheduler can make better decisions about which node to schedule a Pod on if it knows the Pod's resource requirement. The scheduler will only schedule a Pod on a node if the node has enough resources available to satisfy the Pod's total request, which is the sum of the Pod's containers' requests. Limits also reduce resource contention on a node and can make performance more predictable.

You will explore how to declare resource requirements and the effect they have on Pods in this lab step. Note that the cluster has Metrics Server installed in the cluster to view Pod and Node compute resource usage. Metrics Server also allows you to use the kubectl top command to monitor resource utilization for Nodes and Pods.

 

#### Instructions
##### Create a Pod manifest for a Pod that will consume a lot of CPU resources:
```
cat << 'EOF' > load.yaml
apiVersion: v1
kind: Pod
metadata:
  name: load
spec:
  containers:
  - name: cpu-load
    image: cloudacademydevops/stress
    args:
    - -cpus
    - "2"
EOF
```
> The stress image runs a binary that can consume a varying amount of resources. The args instruct stress to attempt to consume two CPU cores, which is how many cores each node in the cluster has.

 

##### Create the Pod:

`kubectl apply -f load.yaml`
> The pod simulates a situation where you notice degraded performance in the cluster. When there are many pods and nodes in the cluster, it is difficult to determine which pod or pods are causing the degradation by inspecting each node and pod individually.

 

##### Display the help document for the top command:

`kubectl top --help`
> The top command is similar to the native Linux top command for measuring CPU and memory usage of processes. You can monitor at the node or pod level.

 

##### List the resource consumption of pods:

`kubectl top pods`

>The load pod is using nearly two full cores (1930milli-cores). Having such an unrestricted pod in your cluster could wreak havoc.

###### Note: It can take a minute for the new pod to appear in the list. If you see an error or do not see the Pod in the list, re-issue the command every 15 seconds until it appears.

 

##### Confirm that the Pod is using almost all the CPU for one of the nodes in the cluster:

`kubectl top nodes`
> Observe one node has a 100% CPU% because each node has only 2 cores.

 

##### Create a similar Pod specification except with CPU and memory resource limits and requests:
```
cat << 'EOF' > load-limited.yaml
apiVersion: v1
kind: Pod
metadata:
  name: load-limited
spec:
  containers:
  - name: cpu-load-limited
    image: cloudacademydevops/stress
    args:
    - -cpus
    - "2"
    resources:
      limits:
        cpu: "0.5" # half a core
        memory: "20Mi" # 20 mebibytes 
      requests:
        cpu: "0.35" # 35% of a core
        memory: "10Mi" # 20 mebibytes
EOF
```
> The resources key is added to specify the limits and requests. The Pod will only be scheduled on a Node with 0.35 CPU cores and 10MiB of memory available. It's important to note that the scheduler doesn't consider the actual resource utilization of the node. Rather, it bases its decision upon the sum of container resource requests on the node. For example, if a container requests all the CPU of a node but is actually 0% CPU, the scheduler would treat the node as not having any CPU available. In the context of this lab, the load Pod is consuming 2 CPUs on a Node but because it didn't make any request for the CPU, its usage doesn't impact following scheduling requests.

 

##### Describe the nodes and output their non-terminated Pods tables to highlight how scheduling decisions are made based on the current workload:

`kubectl describe nodes | grep --after-context=5 "Non-terminated Pods"`
> Notice the Node running the load Pod's table:

Although the Pod is using almost 2 CPUs, the resource table does not allocate any request or limit for the Pod so it does not impact the scheduling decision of the load-limited Pod.

 

##### Create the Pod with the resource constraints:

`kubectl apply -f load-limited.yaml`
 

##### Get the Pods in wide output to display the Node running each Pod:

`kubectl get pods -o wide`

> In this case the load-limited Pod is scheduled on the Node (NODE) that isn't running the load Pod. However, this is only by chance since the load Pod did not make any request for the CPU it is using.

 

##### Wait a minute for the new Pod's metrics to be collected, and then display the Pod resource utilization:

`kubectl top pods`
> Notice the load-limited Pod is using almost half of a CPU core (499milli cores), which is the limit you set. The request is used for making scheduling decisions but the limit impacts the actual utilization. Using requests and limits for CPU and memory can prevent performance issues, and allow the scheduler to make the best use of the cluster's resources.

For completeness, the output for the other possible case when both Pods are scheduled on the same Node is shown below:


In this case, the load-limited Pod is using near half a CPU as specified by its limit. However, the load Pod is now using roughly half a CPU less because it had to give up CPU resources for the load-limited Pod.

To ensure the Pods are scheduled on separate Nodes, a request will be added to the load Pod.

 

##### Update the load Pod manifest to include a request for 1.7 CPU:
```
cat << 'EOF' > load.yaml
apiVersion: v1
kind: Pod
metadata:
  name: load
spec:
  containers:
  - name: cpu-load
    image: cloudacademydevops/stress
    args:
    - -cpus
    - "2"
    resources:
      requests:
        cpu: "1.7"
EOF
```
> The value of 1.7 is chosen because there may not be a Node with a full 2 CPUs available considering some system Pods also make requests in addition to the Pods you create.

 

##### Apply the new Pod manifest:


`kubectl apply --force -f load.yaml`
> The apply command cannot be used without the --force option case because updates are not allowed to update container resources. The --force option first deletes the Pod and then creates a new one.

 

##### Delete the load-limited pod

`kubectl delete pod load-limited`
 

##### Re-create the load-limited pod:
`kubectl apply -f load-limited.yaml`
 

##### Confirm the Pods are running on separate Nodes:

`kubectl get pods -o wide`
> It is now guaranteed that the Pods won't be on the same Node because their combined CPU request is 2.1 and the Nodes only have 2 CPUs.

 

##### Summary
In this lab step, you learned about container resource limits and requests in Kubernetes Pods. Requests help the scheduler decide which nodes to place Pods on and limits help reduce resource contention. The following rules apply when limits or requests are exceeded by Pod containers:

Containers that exceed their memory limits will be terminated and restarted if possible.
Containers that exceed their memory request may be evicted when the node runs out of memory.
Containers that exceed their CPU limits may be allowed to exceed the limit depending on the other Pods on the node. Containers will not be terminated for exceeding CPU limits.