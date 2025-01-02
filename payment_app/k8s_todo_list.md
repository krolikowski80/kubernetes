
## TODO List for Kubernetes Payment Deployment

### 1. Preparation
- [x] Ensure access to a Kubernetes cluster with `kubectl` configured.
- [x] Verify that the `busybox:1.36.0` image is available and accessible from your cluster.

### 2. ConfigMap Tasks
- [x] Create a ConfigMap manifest file (`payment-config.yaml`) to define the `MERCHANT_ID`.
  - Key: `MERCHANT_ID`.
  - Value: `123456`.
- [x] Apply the ConfigMap using `kubectl apply -f payment-config.yaml`.
- [x] Verify the ConfigMap creation with `kubectl get configmap payment-config -o yaml`.

### 3. Secret Tasks
- [x] Encode the `PAYMENT_GATEWAY_API_KEY` value (`k8smaestro`) to Base64.
  - Command: `echo -n "k8smaestro" | base64`.
- [x] Create a Secret manifest file (`payment-secret.yaml`) to store the Base64-encoded key.
  - Key: `PAYMENT_GATEWAY_API_KEY`.
  - Encoded Value: `azhzbWFlc3Rybyo=`.
- [x] Apply the Secret using `kubectl apply -f payment-secret.yaml`.
- [x] Verify the Secret creation with `kubectl get secret payment-secret -o yaml`.

### 4. Deployment Tasks
- [x] Create a Deployment manifest file (`payment-deployment.yaml`) with the following specifications:
  - Image: `busybox:1.36.0`.
  - Command:
    ```bash
    sh -c "echo \"MERCHANT_ID: $MERCHANT_ID\"; echo \"PAYMENT_GATEWAY_API_KEY: $PAYMENT_GATEWAY_API_KEY\""
    ```
  - Environment Variables:
    - `MERCHANT_ID`: From `ConfigMap` key.
    - `PAYMENT_GATEWAY_API_KEY`: From `Secret` key.
- [ ] Apply the Deployment using `kubectl apply -f payment-deployment.yaml`.
- [ ] Verify the Deployment creation with `kubectl get deployment payment-deployment`.

### 5. Testing Tasks
- [ ] Retrieve the pod name for the deployed application:
  ```bash
  kubectl get pods -l app=payment-app
  ```
- [ ] Verify the pod logs to ensure environment variables are correctly loaded:
  ```bash
  kubectl logs <pod-name>
  ```
  - Expected output:
    ```bash
    MERCHANT_ID: 123456
    PAYMENT_GATEWAY_API_KEY: k8smaestro
    ```

### 6. Clean-Up Tasks
- [ ] Delete the Deployment:
  ```bash
  kubectl delete deployment payment-deployment
  ```
- [ ] Delete the ConfigMap:
  ```bash
  kubectl delete configmap payment-config
  ```
- [ ] Delete the Secret:
  ```bash
  kubectl delete secret payment-secret
  ```
