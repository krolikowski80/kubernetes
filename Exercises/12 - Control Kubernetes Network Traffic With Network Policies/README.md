# Configuring Kubernetes Network Policies

:star: **Introduction**
- A Kubernetes network policy declares how a set of pods are allowed to communicate with each other and other network endpoints. By default, pods accept traffic from any source and that poses an obvious security risk. Once a network policy selects a pod, that pod no longer accepts all traffic and instead follows the rules defined in the network policy. The rules can apply to ingress traffic or egress traffic and network policies are namespaced.
<br>
- There is one catch with network policies. You must be using a network provider plugin that supports network policies for network policy resources to have any effect. Examples of container network implementations that do support network policies are Calico (used in this lab) and Romana. Not all container networks support network policies, for example, flannel. The list of container network providers usually mentions if network policies are supported. If security is a concern, you should certainly be using a container network that supports network policies.

In this lab step, you will inspect an existing network policy, and perform tests to see how they behave. You will finish by creating a new network policy.

 

:bulb: **Instructions**
##### 1. Inspect the deny-metadata network policy in the default namespace.

`kubectl get networkpolicy deny-metadata -o yaml`
<br>
:warning: Focus on the spec section:
- The policyTypes list can include one or both of Egress and Ingress depending on what type of traffic the policy applies to. In this case, only Egress is included, so the default ingress behavior of allowing all incoming traffic is not impacted (unless there were other network policies in the namespace that did impact ingress, which is not true in this case). A corresponding egress key is defined. If no egress key was defined, the default behavior would be to block all egress traffic. That is, the rules are whitelists that allow certain traffic and deny all other traffic by default. The egress policy whitelists all outgoing traffic (CIDR notation 0.0.0.0/0) except for the EC2 instance metadata IP address (CIDR notation 169.254.169.254/32). The next instruction will explain the different types of egress policies available.

 

##### 2. Explain the fields available for egress network policies:

`kubectl explain networkpolicy.spec.egress`
> Read through the output to understand the fields. The deny-metadata policy does not specify any ports, so it applies to all ports by default.

 

##### 3. Explain the fields available for the to field of egress network policies:

`kubectl explain networkpolicy.spec.egress.to`

> In addition to the ipBlock used by the deny-metadata policy, you can also select all pods within selected namespaces (namespaceSelector) or pods matching labels in the current namespace (podSelector).

 

##### 4. Explain the fields within an ipBlock field of egress network policies:

`kubectl explain networkpolicy.spec.egress.to.ipBlock`
> You will test the network policy in the following instructions.

 

##### 5. Create a pod in the default namespace and run /bin/sh in it:

`kubectl run busybox --image=busybox --rm -it /bin/sh`
> After a short delay, you will see the following shell prompt:

##### 6. Use the wget command to connect to https://google.com:

`wget https://google.com`
> The command succeeds in connecting and downloading the index.html page.

 

##### 7. Attempt to connect to the EC2 instance metadata endpoint:

`wget 169.254.169.254`

> The command will never connect to the endpoint because of the deny-metadata network policy.

 

##### 8. Press ctrl+c to cancel the command, and enter exit to stop the pod:

`exit`
 

##### 9. Retry connecting to the EC2 instance metadata endpoint using a container in a different namespace:

`kubectl create namespace test`
`kubectl run busybox --image=busybox --rm -it -n test /bin/sh`
`wget 169.254.169.254`
> altThis time the connection succeeds. The EC2 instance metadata endpoint can potentially be used by malicious containers to extract credentials and modify resources in your AWS account. That is why it is important to prevent access to the metadata endpoint.

 

##### 10. Extract AWS credentials using the metadata endpoint:

`role=$(wget -qO- 169.254.169.254/latest/meta-data/iam/security-credentials)`
`wget -qO- 169.254.169.254/latest/meta-data/iam/security-credentials/$role`

> Depending on the IAM permissions of the instance role, the credentials could be used to do a lot of damage. Configuring the deny-metadata network policy to apply to all namespaces is much safer. Also, using a private image registry that only contains trusted images can help limit the risk of malicious code entering your cluster.

 

##### 11. Terminate the pod:

`exit`
 

##### 12. Create a network policy resource file that uses pod selectors to allow incoming connections to a web server application tier from a cache tier:
```
cat > app-policy.yaml <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-tiers
  namespace: test
spec:
  podSelector:
    matchLabels:
      app-tier: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app-tier: cache
    ports:
    - port: 80
EOF
```
> The network policy includes an ingress policy that uses a podSelector. The format is similar to egress policies, but uses a from field instead of a to field.

 

##### 13. Create the network policy resource:

`kubectl create -f app-policy.yaml`
 

##### 14. Create an Nginx web server pod in the web server tier:

`kubectl run web-server -n test -l app-tier=web --image=nginx:1.15.1 --port 80`

##### 15. Create a busybox pod in the cache tier to test the app-tiers network policy:

###### Get the web server pod's IP address
`
web_ip=$(kubectl get pod -n test -o jsonpath='{.items[0].status.podIP}')`
`kubectl run busybox -n test -l app-tier=cache --image=busybox --env="web_ip=$web_ip" --rm -it /bin/sh
`
`wget $web_ip`
`exit`

> The request succeeds. You could also leverage the Kubernetes DNS to resolve the web server's pod DNS name to an IP address. However, you need the IP address to construct the default pod DNS name, so the IP address is used directly.

 

##### 16. Repeat the test, but using a pod that is not in the cache tier:

`kubectl run busybox -n test --image=busybox --env="web_ip=$web_ip" --rm -it /bin/sh`
`wget $web_ip`

> The request will never succeed.

 

##### 17. Press ctrl+c to cancel the command, and enter exit to stop the pod:

`exit`
 

#### Summary
In this lab step, you learned how network policies can be used to control network access to pods in Kubernetes. Network policies can be declared using a mixture of namespace selectors, pod selectors, and IP blocks for both ingress and egress.
