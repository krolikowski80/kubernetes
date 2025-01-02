
# Wdrożenie aplikacji Shipy w Google Cloud Platform (GCP)

## Przygotowanie środowiska
1. **Zaloguj się do konsoli GCP**: [https://console.cloud.google.com](https://console.cloud.google.com).
2. **Utwórz nowy projekt lub wybierz istniejący**.
3. **Włącz API Kubernetes Engine**:
   - Przejdź do **API & Services > Library**.
   - Wyszukaj **Kubernetes Engine API** i włącz je.
4. **Utwórz klaster GKE**:
   - Przejdź do **Kubernetes Engine > Clusters**.
   - Kliknij **Create Cluster**.
   - Wybierz typ klastra (np. Standard) i skonfiguruj:
     - Liczba węzłów: 1–3 (dla testów wystarczy 1).
     - Region: wybierz najbliższy dla Twojej lokalizacji.
   - Kliknij **Create**.

## Konfiguracja `kubectl`
1. Pobierz i zainstaluj **Google Cloud SDK**: [https://cloud.google.com/sdk](https://cloud.google.com/sdk).
2. Zaloguj się do swojego projektu:
   ```bash
   gcloud auth login
   gcloud config set project [PROJECT_ID]
   ```
3. Skonfiguruj `kubectl`:
   ```bash
   gcloud container clusters get-credentials [CLUSTER_NAME] --region [REGION]
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
