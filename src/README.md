## Some simple tasks you can perform using kubectl:


#### Multi-Container Pods
```
kubectl create -f 3.1-namespace.yaml
kubectl create -f 3.2-multi_container.yaml -n microservice
kubectl get -n microservice pod app
kubectl describe -n microservice pod app
kubectl logs -n microservice app counter --tail 10
kubectl logs -n microservice app poller -f
 ```

#### Service Discovery
```kubectl create -f 4.1-namespace.yaml
kubectl create -f 4.2-data_tier.yaml -n service-discovery
kubectl get pod -n service-discovery
kubectl describe service -n service-discovery data-tier
kubectl create -f 4.3-app_tier.yaml -n service-discovery
kubectl create -f 4.4-support_tier.yaml -n service-discovery
kubectl get pods -n service-discovery
kubectl logs -n service-discovery support-tier poller -f
```

#### Deployments
```kubectl create -f 5.1-namespace.yaml
kubectl create -n deployments -f 5.2-data_tier.yaml -f 5.3-app_tier.yaml -f 5.4-support_tier.yaml
kubectl get -n deployments deployments
kubectl -n deployments get pods
kubectl scale -n deployments deployment support-tier --replicas=5
kubectl -n deployments get pods
kubectl delete -n deployments pods support-tier-... support-tier-... --wait=false (You can use tab completion to display the possible values to replace ... with)
watch -n 1 kubectl -n deployments get pods
kubectl scale -n deployments deployment app-tier --replicas=5
kubectl -n deployments get pods
kubectl describe -n deployments service app-tier
``` 

#### Autoscaling
```kubectl apply -f metrics-server/ # apply recent version of metrics server
watch kubectl top pods -n deployments
kubectl create -f 6.1-app_tier_cpu_request.yaml -n deployments
kubectl apply -f 6.1-app_tier_cpu_request.yaml -n deployments
kubectl get -n deployments deployments app-tier
kubectl create -f 6.2-autoscale.yaml -n deployments
watch -n 1 kubectl get -n deployments deployments app-tier # This can take up to 10 minutes to fully take effect mainly due to the autoscaler's 5 minute default initialization period. You may want to skip waiting. The image below shows the output of kubectl describe -n deployments hpa after waiting long enough
alt
kubectl api-resources
kubectl describe -n deployments hpa
kubectl get -n deployments hpa
kubectl edit -n deployments hpa
watch -n 1 kubectl get -n deployments deployments app-tier
 ```

#### Rolling Updates and Rollbacks
```kubectl delete -n deployments hpa app-tier
kubectl edit -n deployments deployment app-tier
watch -n 1 kubectl get -n deployments deployments app-tier # To watch this log, you should use a multi-terminal tool like tmux.
kubectl edit -n deployments deployment app-tier
kubectl rollout -n deployments status deployment app-tier
kubectl edit -n deployments deployments app-tier (left terminal)
kubectl rollout -n deployments status deployment app-tier (right terminal)
kubectl rollout -n deployments pause deployment app-tier (left terminal)
kubectl get deployments -n deployments app-tier (left terminal)
kubectl rollout -n deployments resume deployment app-tier (left terminal)
kubectl rollout -n deployments undo deployment app-tier
kubectl scale -n deployments deployment app-tier --replicas=1
```

#### Probes
```kubectl create -f 7.1-namespace.yaml
kubectl create -f 7.2-data_tier.yaml -n probes
kubectl get deployments -n probes -w
kubectl create -f 7.3-app_tier.yaml -n probes
kubectl get -n probes deployments app-tier -w # -w: The -w option (short for --watch) allows you to continuously watch for changes in real-time. The command will display live updates as the deployment status changes.
kubectl get -n probes pods
kubectl logs -n probes app-tier-... | cut -d' ' -f5,8-11 (You can use tab completion to display the possible values to replace ... with)
``` 

#### Init Containers
```kubectl apply -f 8.1-app_tier.yaml -n probes
kubectl describe pod -n probes app-tier... (You can use tab completion to display the possible values to replace ... with)
kubectl logs -n probes app-tier-... await-redis (You can use tab completion to display the possible values to replace ... with)
 ```

#### Volumes
```kubectl -n deployments logs support-tier-... poller --tail 1 (You can use tab completion to display the possible values to replace ... with)
kubectl exec -n deployments data-tier-... -it -- /bin/bash (You can use tab completion to display the possible values to replace ... with)
kill 1
kubectl -n deployments get pods
kubectl -n deployments logs support-tier-... poller --tail 1 (You can use 
tab completion to display the possible values to replace ... with)
```
##### Note: It takes around a couple of minutes for the effects of the restart to settle. The poller will stop updating and report the last value before restarting until it can reach the new data tier value. Try again after a minute if you don't see a relatively small value)
```kubectl create -f 9.1-namespace.yaml
aws ec2 describe-volumes --region=us-west-2 --filters="Name=tag:Type,Values=PV" --query="Volumes[0].VolumeId" --output=text
vol_id=$(aws ec2 describe-volumes --region=us-west-2 --filters="Name=tag:Type,Values=PV" --query="Volumes[0].VolumeId" --output=text)
sed -i "s/INSERT_VOLUME_ID/$vol_id/" 9.2-pv_data_tier.yaml
kubectl create -n volumes -f 9.2-pv_data_tier.yaml -f 9.3-app_tier.yaml -f 9.4-support_tier.yaml
kubectl describe -n volumes pvc
kubectl describe -n volumes pod data-tier-... (You can use tab completion to display the possible values to replace ... with)
kubectl logs -n volumes support-tier-... poller --tail 1 (You can use tab completion to display the possible values to replace ... with)
```
##### Note: It takes a few minutes for all of the readiness checks to pass and for the counter to start incrementing. If you don't see a counter value output then try again after a minute or two.
```kubectl delete -n volumes deployments data-tier
kubectl get -n volumes pods
kubectl create -n volumes -f 9.2-pv_data_tier.yaml
kubectl logs -n volumes support-tier-... poller --tail 1 (You can use tab completion to display the possible values to replace ... with)
 ```

#### Config Maps and Secrets
```kubectl create -f 10.1-namespace.yaml
kubectl create -n config -f 10.2-data_tier_config.yaml -f 10.3-data_tier.yaml
kubectl exec -n config data-tier-... -it -- /bin/bash (You can use tab completion to display the possible values to replace ... with)
cat /etc/redis/redis.conf
redis-cli CONFIG GET tcp-keepalive
exit
kubectl edit -n config configmaps redis-config
kubectl exec -n config data-tier-... -- redis-cli CONFIG GET tcp-keepalive (You can use tab completion to display the possible values to replace ... with)
```
##### This command is used in Kubernetes to restart the deployment named data-tier in the config namespace. Here’s a breakdown of each part of this command:
`
kubectl rollout -n config restart deployment data-tier`
- kubectl: Command-line tool for managing and interacting with Kubernetes clusters.
- rollout: Command for managing application rollout strategies.
- -n config: Specifies that the operation should be performed in the config namespace.
- restart deployment data-tier: Forces a restart of the deployment named data-tier.
- ###### In summary, this command is used to initiate a restart of the data-tier deployment specifically within the config namespace in Kubernetes.

```kubectl exec -n config data-tier-... -- redis-cli CONFIG GET tcp-keepalive (You can use tab completion to display the possible values to replace ... with)
kubectl create -f 10.4-app_tier_secret.yaml -n config
kubectl describe -n config secret app-tier-secret
kubectl edit -n config secrets app-tier-secret
kubectl create -f 10.5-app_tier.yaml -n config
kubectl exec -n config app-tier-... -- env (You can use tab completion to display the possible values to replace ... with)
```