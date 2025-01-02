
## Kubernetes Deployment for Payment Processing Application

### 1. ConfigMap Definition
Create a ConfigMap that contains the `MERCHANT_ID`.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: payment-config
  namespace: default
data:
  MERCHANT_ID: "123456"
```

### 2. Secret Definition
Create a Secret that contains the `PAYMENT_GATEWAY_API_KEY`. The value should be encoded in Base64.

To encode the value `k8smaestro` in Base64:
```bash
echo -n "k8smaestro" | base64
```
This outputs:
```
azhzbWFlc3Rybyo=
```

Create the Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: payment-secret
  namespace: default
data:
  PAYMENT_GATEWAY_API_KEY: "azhzbWFlc3Rybyo="
```

### 3. Deployment Definition
Create a Deployment that uses the ConfigMap and Secret to pass environment variables to the Busybox container.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: payment-app
  template:
    metadata:
      labels:
        app: payment-app
    spec:
      containers:
      - name: busybox-payment
        image: busybox:1.36.0
        command:
        - sh
        - -c
        - |
          echo "MERCHANT_ID: $MERCHANT_ID"
          echo "PAYMENT_GATEWAY_API_KEY: $PAYMENT_GATEWAY_API_KEY"
        env:
        - name: MERCHANT_ID
          valueFrom:
            configMapKeyRef:
              name: payment-config
              key: MERCHANT_ID
        - name: PAYMENT_GATEWAY_API_KEY
          valueFrom:
            secretKeyRef:
              name: payment-secret
              key: PAYMENT_GATEWAY_API_KEY
```

### 4. Steps to Deploy and Test

#### 4.1 Apply the ConfigMap, Secret, and Deployment
Run the following commands to create the Kubernetes objects:
```bash
kubectl apply -f payment-config.yaml
kubectl apply -f payment-secret.yaml
kubectl apply -f payment-deployment.yaml
```

#### 4.2 Verify the Deployment Logs
Get the pod name:
```bash
kubectl get pods -l app=payment-app
```

Check the logs of the pod:
```bash
kubectl logs <pod-name>
```

Expected output:
```
MERCHANT_ID: 123456
PAYMENT_GATEWAY_API_KEY: k8smaestro
```

### 5. Clean Up Resources
To delete all resources created:
```bash
kubectl delete deployment payment-deployment
kubectl delete configmap payment-config
kubectl delete secret payment-secret
```
