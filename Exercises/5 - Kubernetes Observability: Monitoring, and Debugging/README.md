# Using Probes to Better Understand Pod Health
### Introduction
* If a Pod is in a Running state it does not necessarily mean the Pod is ready to use or that the Pod is operating as you would expect. For example, a running Pod may not be ready to accept requests or may have entered into an internal failed state, such as a deadlock, preventing it to make progress on new requests. Kubernetes introduces probes for containers to detect when Pods are in such states. More specifically:

* Readiness probes are used to detect when a Pod is unable to serve traffic, such as during startup when large amounts of data are being loaded or caches are being warmed. When a readiness probe fails, it means that the Pod needs more time to become ready to serve traffic. When the Pod is accessed through a Kubernetes Service, the Service will not serve traffic to any Pods that have a failing readiness probe. 
* Liveness probes are used to detect when a Pod fails to make progress after entering a broken state, such as deadlock. The issues causing the Pod to enter such a broken state are bugs, but by detecting a Pod is in a broken state allows Kubernetes to restart the Pod and allow progress to be made, perhaps only temporarily until arriving at the broken state again. The application availability is improved compared to leaving the Pod in the broken state.
* Startup probes are used when an application starts slowly and may otherwise be killed due to failed liveness probes. The startup probe runs before both readiness and liveness probes. The startup probe can be configured with a startup time that is longer than the time needed to detect a broken state for a container after it has started.
* Readiness and Liveness probes run for the entire lifetime of the container they are declared in. Startup probes only run until they first succeed. A container can define up to one of each type of probe. All probes are also configured the same way. The only difference is how a probe failure is interpreted.

* You will use both Readiness and Liveness probes in this lab step.

 

#### Instructions
##### Explain the readinessProbe field which is at the path pod.spec.containers.readinessProbe:

`kubectl explain pod.spec.containers.readinessProbe`

> Read through the DESCRIPTION and FIELDS. There are three types of actions a probe can take to assess the readiness of a Pod's container:

*  exec: Issue a command in the container. If the exit code is zero the container is a success, otherwise it is a failed probe.
httpGet: Send a HTTP GET request to the container at a specified path and port. If the HTTP response status code is a 2xx or 3xx then the container is a success, otherwise, it is a failure.
tcpSocket: Attempt to open a socket to the container on a specified port. If the connection cannot be established, the probe fails.
The number of consecutive successes is configured via the successThreshold field and the number of consecutive failures required to transition from success to failure is failureThreshold. The probe runs every periodSeconds and each probe will wait up to timeoutSeconds to complete.

 

##### Create a Pod that uses a readinessProbe with an HTTP GET action:
```
cat << 'EOF' > pod-readiness.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-http
spec:
  containers:
  - name: readiness
    image: httpd:2.4.38-alpine
    ports:
    - containerPort: 80
    # Sleep for 30 seconds before starting the server
    command: ["/bin/sh","-c"]
    args: ["sleep 30 && httpd-foreground"]
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 3
EOF
kubectl create -f pod-readiness.yaml
```

> The probe will check for a successful HTTP response from the Pod's IP on port 80. The container command delays starting the server by sleeping for 30 seconds to simulate an intense startup routine. You can also set custom HTTP headers for the probe, issue kubectl explain pod.spec.containers.readinessProbe.httpGet for details.

 

##### Describe the Pod to see how events related to the readiness probe:

`kubectl describe pod readiness-http`
>In the Events you should see some Warning entries related to the failed probe:
There will be about 10 (x10) failed probes since the container sleeps for 30 seconds and the probe runs every 3 seconds after an initial 3-second delay. You can see when the probe succeeds by looking at the Conditions section:

* The Ready and ContainerReady Status will both be True once the probe succeeds. They will be False until then. Also, note the Containers section summarizes the configured readiness probe:

##### To confirm the readiness probes are always running, kill the httpd server processes running in the container

`kubectl exec readiness-http -- pkill httpd`
>This command runs pkill httpd (kill all httpd processes) inside the readiness-http Pod's only container. The two dashes (--) are used to indicate where the command to run begins and options parsing for kubectl exec ends.

 

##### Describe the Pod and observe the Ready Conditions are False:

`kubectl describe pod readiness-http`
>Note: The httpd server processes will recover from being killed after a minute. If more than a minute has passed since you issued the previous instruction's command, you may see both conditions are True.  If that happened, you can kill the httpd processes again and quickly describe the Pod.

 

##### Create a Pod that uses a liveness probe to detect broken states:
```
cat << 'EOF' > pod-liveness.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-tcp
spec:
  containers:
  - name: liveness
    image: busybox:1.30.1
    ports:
    - containerPort: 8888
    # Listen on port 8888 for 30 seconds, then sleep
    command: ["/bin/sh", "-c"]
    args: ["timeout 30 nc -p 8888 -lke echo hi && sleep 600"]
    livenessProbe:
      tcpSocket:
        port: 8888
      initialDelaySeconds: 3
      periodSeconds: 5
EOF
kubectl create -f pod-liveness.yaml
```
>Recall that liveness probes have the same configuration as readiness probes. The nc (netcat) command listens (the -l option) for connections on port (-p) 8888 and responds with the message hi for the first 30 seconds, after which timeout kills the nc server. The liveness probe attempts to establish a connection with the server every 5 seconds.

 

##### Watch the describe output for the Pod to observe when the liveness probe fails and the Pod is restarted:

`watch kubectl describe pod liveness-tcp`
>The Events will list BackOff and Unhealthy events when the liveness probe fails. Over time you should see the Restart Count increment about every minute:


* Recall that the probe needs to fail three times after having success to consider the probe as failed.

* Press ctrl+c to stop watching the output.

 

##### Delete the Pods created in this lab step:
```
kubectl delete -f pod-readiness.yaml
kubectl delete -f pod-liveness.yaml
```
 

### Summary
* In this lab step, you understood the purpose of and learned how to use both readiness and liveness probes. Readiness probes allow you to detect when a Pod is not ready to serve traffic and Kubernetes should wait longer for the Pod to become ready. Liveness probes allow you to detect when a Pod has entered a broken state and is restarted automatically. Startup probes can be used to separate the probe behavior at startup from the probe behavior post-startup.


----------
### Monitoring Kubernetes Applications
#### Introduction
* There are many metrics that you may want to keep track of in Kubernetes: available node CPU/Memory, number of running Pods vs. desired Pods for Deployments, errors and warnings, etc. There are several built-in mechanisms available to make monitoring and debugging applications easier. You can also leverage purpose-built monitoring systems that run in Pods. Kubernetes has its own basic monitoring solution called Metrics Server. You will use Metrics Server later in this lab step. You can also use more complete metrics pipelines, such as Prometheus, but that is beyond the scope of this lab. 

 
#### Instructions
##### Use the get command to ensure all Pods have all containers running:

`kubectl get pods --all-namespaces`

> The READY column separates the number of containers that are ready and the number of containers in a Pod by a slash. You can quickly get a view of all the Pods in the cluster by adding the --all-namespaces option to get. Similarly, for any other Resource type, you can use get to quickly get a view of those Resources. For example, the output for Deployments lists the desired and actual number of Pods that are in each Deployment.

* To get more information you can use some commands you have used extensively in this Lab: 

- describe which includes information about the resource itself as well as related resources and events
- logs for diagnosing any failures related to Pods and containers

- The get command can also be used to view complete configuration details by adding instructing the command to output in YAML format (-o yaml). This can help you diagnose any suspected configuration issues.

 
##### List all the events in the default namespace:
`kubectl get events -n default`

> You can retrieve events for individual resources by using the describe command, but the get events command allows you to see all events in sequence. Events are namespaced so you can limit your search by Namespace. To get additional information, you can instruct get to output in wide format (-o wide).

 

##### Enter the following command to see how to use the top command to monitor node and Pod resource utilization:

`kubectl top`

> The kubectl top command is in the same spirit as the top command for Unix-based operating systems. As the output states, the top command depends on a properly configured system for collecting the usage information. Metrics Server implements the Kubernetes metrics API. Other metric pipelines can also implement the metrics API to work with top and provide additional functionality, e.g. Prometheus, however, they are outside of the scope of this lab.

 

##### Use top to view resource utilization metrics for the cluster's nodes:

`kubectl top node`
> The output displays CPU usage in cores and %. The master and nodes both have 2 CPU cores available. The CPU(cores) column would have a maximum value of 2000m (2000 milliCPU cores = 2 CPU cores). The CPU metrics indicate that the cluster is not under CPU pressure. The MEMORY columns indicate that there is greater than 50% memory available on the nodes. That indicates the cluster if oversized for the current workload. As new Pods are added you should monitor the memory pressure. You should also set CPU and memory limits and requests in your Pod containers to avoid an unexpected degradation in performance. Issue kubectl explain pod.spec.containers.resources for more information. 

 

##### Use top to view resource utilization metrics for the Pods in the kube-system Namespace:
`kubectl top pods -n kube-system`
> From this view, you can see if there are any indications that a Pod is using an unexpected amount of resources.

 

##### Display the resource utilization of individual containers:

`kubectl top pod -n kube-system --containers`
>The --containers option allows you to understand the resource utilization at a finer granularity when Pods contain multiple containers. In this example, all Pods are only running a single container, however. The NAME column refers to container names.

 

##### Use a label selector to show only resource utilization for Pods with a k8s-app=kube-dns label:

`kubectl top pod -n kube-system --containers -l k8s-app=kube-dns`
> Label selectors make it easy to focus your search as long as you have a well-defined labeling strategy.

 

#### Summary
- In this lab step, you used several kubectl commands that allow you to monitor your Kubernetes applications. You used Metrics Server to implement the metrics API that the top command depends on.

- If you have more time in your lab session, try issuing more logging, monitoring, and debugging types of commands. Try to inspect Pods, Deployments, and Services in the kube-system Namespace. For example:

- See if you can identify which container uses the most CPU or memory in the kube-system namespace.
See if you can get the Debian version used by kube-proxy Pods from their /etc/debian_version file.