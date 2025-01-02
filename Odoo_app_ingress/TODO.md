
# TODO – Rozwiązywanie problemów

## **1. Weryfikacja Ingress Controller**

- Upewnij się, że Ingress Controller działa poprawnie:
  ```bash
  kubectl get pods -n ingress-nginx
  ```

- Sprawdź logi kontrolera:
  ```bash
  kubectl logs -n ingress-nginx <pod-name>
  ```

## **2. Sprawdzenie konfiguracji Service**

- Zweryfikuj, czy Service dla aplikacji Odoo jest poprawnie skonfigurowany:
  ```bash
  kubectl get service -n odoo
  ```

- Przykład poprawnego Service:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: odoo-service
    namespace: odoo
  spec:
    selector:
      app: odoo
    ports:
      - protocol: TCP
        port: 8069
        targetPort: 8069
    type: ClusterIP
  ```

## **3. Testowanie lokalne za pomocą port-forward**

Jeśli aplikacja nie działa przez Ingress, przetestuj ją lokalnie:
```bash
kubectl port-forward svc/odoo-service -n odoo 8069:8069
```
Następnie otwórz przeglądarkę i przejdź do `http://localhost:8069`.

## **4. Weryfikacja DNS i hosta**

- Jeśli używasz hosta (np. `odoo.example.com`), upewnij się, że DNS jest skonfigurowany poprawnie.
- Dodaj wpis do pliku `/etc/hosts` na swoim komputerze (dla testów):
  ```text
  <public-ip-address> odoo.example.com
  ```

## **5. Debugowanie Ingress**

- Sprawdź status obiektu Ingress:
  ```bash
  kubectl describe ingress odoo-ingress -n odoo
  ```

- Zweryfikuj logi Ingress Controller:
  ```bash
  kubectl logs -n ingress-nginx <pod-name>
  ```

## **6. Usuwanie zasobów**

W przypadku konieczności wyczyszczenia środowiska:
```bash
kubectl delete ingress odoo-ingress -n odoo
kubectl delete service odoo-service -n odoo
kubectl delete deployment odoo -n odoo
```
