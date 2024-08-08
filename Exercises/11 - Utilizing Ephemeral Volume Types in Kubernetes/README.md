# Utilizing Ephemeral Volume Types in Kubernetes

:star: **Introduction**
- In Kubernetes, ephemeral volumes have their lifetime tied to the lifetime of the Pod they are mounted in. Once the Pod is deleted, so is the volume and its data. It is important to note that the data does survive across container restarts.

<br>

- There are a variety of use cases for ephemeral volumes, including:
  - Sharing data between containers in a Pod (multi-container Pods)
  - Providing read-only input configuration data to a Pod
  - Temporary data caches
  - Providing read-only configuration data to Pods is best suited by using Kubernetes ConfigMaps and Secrets. When ConfigMaps and Secrets are mounted as volumes, the data is stored in an ephemeral volume. Secrets and ConfigMaps are the subject of other content and won't be discussed further in this lab.

<br>

- You will understand how Kubernetes provides ephemeral storage and understand how to utilize ephemeral storage in this lab step.

 
<br>

:bulb: **Instructions**
##### 1. Create a Namespace for the resources you'll create in this lab step and change your default kubectl context to use the Namespace:

###### Create namespace
`kubectl create namespace ephemeral`
###### Set namespace as the default for the current context
`kubectl config set-context $(kubectl config current-context) --namespace=ephemeral`
##### 2. Use an emptyDir volume to store the logs of a Pod's container that is recording random coin tosses:
```
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: coin-toss
spec:
  containers:
  - name: coin-toss
    image: busybox:1.33.1
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true;
      do
        # Record coint tosses
        if [[ $(($RANDOM % 2)) -eq 0 ]]; then echo Heads; else echo Tails; fi >> /var/log/tosses.txt;
        sleep 1;
      done
    # Mount the log directory /var/log using a volume
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  # Declare log directory volume an emptyDir ephemeral volume
  volumes:
  - name: varlog
    emptyDir: {}
EOF
```

> The fields for creating a volume (spec.volumes) and mounting the volume in the container (spec.containers.volumeMounts) are independent of whether you use ephemeral or persistent volumes. The only way to know the Pod is using an ephemeral volume is by seeing the volume's emptyDir field. In the manifest the braces (emptyDir: {}) symbolize an empty object because the emptyDir is using default configuration values. The emptyDir volume type creates an empty directory for use by the Pod and shares the lifetime of the Pod, i.e. it is ephemeral. The directory is created on the Node's file system by default and can therefore survive node restarts (assuming the Pod remains scheduled on the Node).

 

##### 3. List the contents of the volume on the Node that is running the Pod:

`pod_node=$(kubectl get pod coin-toss -o jsonpath='{.status.hostIP}')`

`pod_id=$(kubectl get pod coin-toss -o jsonpath='{.metadata.uid}')`

`ssh $pod_node -oStrictHostKeyChecking=no`

`sudo ls /var/lib/kubelet/pods/$pod_id/volumes/kubernetes.io~empty-dir/varlog`

> The volume starts empty but now has the tosses.txt file which logs the coin toss results for the Pod. You would not usually access the volume contents this way. You will use kubectl exec to access the file in the coming instructions.

- If you are concerned about storing potentially sensitive data in a volume that is accessible on a Node's file system, you can configure emptyDir to use memory rather than disk storage.

 

##### 4. Explain the emptyDir Volume object to view its configuration fields:

`kubectl explain pod.spec.volumes.emptyDir`
> The medium can be configured to use tmpfs memory-based storage by setting medium: Memory. This will result in higher performance than using disk-backed volumes. However, the memory used does counts against container memory limits. Note that when using memory (RAM) the data will not survive Node restarts in contrast to using disk. 

 
##### 5. exec into the Pod's container to count how many lines are in the /var/log/tosses.txt file using the wc -l command: 

`kubectl exec coin-toss -- wc -l /var/log/tosses.txt`
> You will now verify that the data on the ephemeral volume survives Pod container restarts.

 

###### 6. set the Pod to use a new image and wait for the Pod to restart its container:

`kubectl set image pod coin-toss coin-toss=busybox:1.34.0 kubectl get pods -w`
:bulb: It takes under a minute to restart.

 

##### 7. Press ctrl+c to stop watching the Pod and re-issue the exec command to verify the count didn't restart from zero:

`kubectl exec coin-toss -- wc -l /var/log/tosses.txt`
> This confirms the ephemeral volume is tied to the Pod's lifetime, not the container's. If the ephemeral volume had not been used and only the container's writable layer storage was used, the data would have been lost on restarting the container. Your application requirements will dictate if that is acceptable. A volume should be used if it is preferable to tie the storage to the Pod's lifetime and not a container's lifetime.

 

##### 8. Delete the Pod:

`kubectl delete pod coin-toss`

 

##### 9. Confirm the volume has been deleted by attempting to list the Pod's volume's contents on the Node hosting the Pod:

`ssh $pod_node -oStrictHostKeyChecking=no`
`sudo ls /var/lib/kubelet/pods/$pod_id/volumes/kubernetes.io~empty-dir/varlog`
> This confirms the data is lost once the Pod is deleted.

- Because ephemeral storage is finite on each Node, you can configure requests and limits for it, just as you can for CPU and memory. You will see how to configure the ephemeral storage request and limit for a Pod next.

 

##### 10. Create a Pod that uses an emptyDir ephemeral volume for its data and requests and limits the Pod's total ephemeral storage (ephemeral-storage) to the very low value of 1 kibibyte (1Ki):
```
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: cache
spec:
  containers:
  - name: cache
    image: redis:6.2.5-alpine
    resources:
      requests:
        ephemeral-storage: "1Ki"
      limits:
        ephemeral-storage: "1Ki"
    volumeMounts:
    - name: ephemeral
      mountPath: "/data"
  volumes:
    - name: ephemeral
      emptyDir:
        sizeLimit: 1Ki
EOF
```
> The resources object is the same one used for CPU and memory although only ephemeral-storage is declared in this example. The requests value is used to ensure the Node that runs the Pod's container has enough ephemeral storage available to run the Pod. If no Nodes can fulfill the request, the Pod is left in a pending state. The limit sets the maximum allowed ephemeral storage for the Pod. You will observe what happens when the limit is exceeded next.

 

##### 11. Describe the Pod and see what the result of the low ephemeral storage limit is:

`kubectl describe pod cache`
> The Pod is Scheduled onto a Node because there is a Node in the cluster satisfying the ephemeral storage request. After the Pod is Started, its container exceeds the ephemeral storage limit set (Usage of EmptyDir volume "ephemeral" exceeds the limit "1Ki"). The kubelet then evicts the Pod and it is subsequently killed. Taking time to understand the storage requirements of your Pods can avoid eviction while giving you the confidence that whatever Node a Pod is running on has the required ephemeral storage available for it.

 

#### Summary
In this lab step, you utilized ephemeral volumes to store log and cache data. These are two applications where it makes sense to have the lifetime of the data connected to the lifetime of the associated Pod. You also learned about emptyDir volumes, a common ephemeral volume type. You saw where the volume's data is physically stored and understood the intricacies of the data's lifetime. Lastly, you learned how to provide requests for ephemeral storage to guarantee a Node has enough for a Pod and to limit a Pod's ephemeral storage to prevent a Pod from hogging storage resources of a Node.
