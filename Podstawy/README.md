## Podstawowe polecenia

#### usuwanie poda
``
kubectl delete pod <pod-name>
``

#### wykonanie polecenia w kontenerze
``
kubectl exec -it <pod-name> -c <container-name> <command>
``

#### czytanie logów kontenera
``
kubectl logs <pod-name> -c <container-name> -f 
``

#### deployment poda
``
kubectl apply -f <manifest-name.yml>
``

#### przegląd poda
``
kubectl describe pod <pod-name>
``