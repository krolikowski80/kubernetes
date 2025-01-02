
# Wdrożenie aplikacji Shipy w Amazon Web Services (AWS)

## Przygotowanie środowiska
1. **Zaloguj się do konsoli AWS**: [https://aws.amazon.com](https://aws.amazon.com).
2. **Przejdź do usługi EKS**.
3. **Utwórz klaster**:
   - Kliknij **Create Cluster**.
   - Wprowadź nazwę klastra.
   - Wybierz wersję Kubernetes (np. 1.24 lub wyższą).
   - Skonfiguruj sieć (VPC) i podłącz podsieci.
   - Kliknij **Create**.
4. **Utwórz grupę węzłów**:
   - Przejdź do **Compute > Node Groups**.
   - Kliknij **Add Node Group**.
   - Wybierz typ instancji (np. t3.medium) i liczbę węzłów.

## Konfiguracja `kubectl`
1. Pobierz i zainstaluj **AWS CLI**: [https://aws.amazon.com/cli/](https://aws.amazon.com/cli/).
2. Zainstaluj narzędzie **eksctl**: [https://eksctl.io/](https://eksctl.io/).
3. Połącz `kubectl` z klastrem:
   ```bash
   aws eks update-kubeconfig --region [REGION] --name [CLUSTER_NAME]
   ```

## Wdrożenie aplikacji Shipy
1. **Przygotowanie namespace**:
   Upewnij się, że namespace `shipy` istnieje:
   ```bash
   kubectl create namespace shipy
   ```
2. **Wdrażanie obiektów YAML**:
   - Utwórz pliki `Deployment`, `Service` i `Ingress` zgodnie z przygotowanymi szablonami.
   - Wdróż je do klastra:
     ```bash
     kubectl apply -f deployment.yaml -n shipy
     kubectl apply -f service.yaml -n shipy
     kubectl apply -f ingress.yaml -n shipy
     ```
3. **Sprawdzenie działania**:
   - Pobierz publiczny adres IP Ingress Controller:
     ```bash
     kubectl get ingress -n shipy
     ```
   - Otwórz adres w przeglądarce.

## Debugowanie
1. **Sprawdzenie logów**:
   ```bash
   kubectl logs deployment/shipy-webapp -n shipy
   ```
2. **Port-forward** (opcjonalnie):
   ```bash
   kubectl port-forward service/shipy-webapp-service 7000:7000 -n shipy
   ```
