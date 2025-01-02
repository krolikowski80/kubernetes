
# TODO Lista dla wdrożenia systemu Odoo w Kubernetes

## 1. Przygotowanie środowiska
- [x] Upewnij się, że masz działający klaster Kubernetes.
- [x] Zainstaluj narzędzie `kubectl` i sprawdź połączenie z klastrem (`kubectl get nodes`).
- [ ] Utwórz namespace `odoo` w Kubernetes:
  ```bash
  kubectl create namespace odoo
  ```

## 2. Wdrożenie bazy danych PostgreSQL
- [ ] Pobierz plik `postgres-deployment.yaml`:
  [postgres-deployment.yaml](https://raw.githubusercontent.com/dnaprawa/KubernetesMaestro/main/02/self-study/postgres-deployment.yaml)
- [ ] Zmień namespace w pliku YAML na `odoo`.
- [ ] Zaaplikuj plik YAML do klastra:
  ```bash
  kubectl apply -f postgres-deployment.yaml -n odoo
  ```
- [ ] Zweryfikuj, czy Pod został uruchomiony poprawnie:
  ```bash
  kubectl get pods -n odoo
  ```

## 3. Stworzenie obiektu Service dla PostgreSQL
- [ ] Utwórz plik `postgres-service.yaml` z konfiguracją Service typu `ClusterIP` dla Postgresa.
- [ ] Zaaplikuj plik YAML do klastra:
  ```bash
  kubectl apply -f postgres-service.yaml -n odoo
  ```

## 4. Wdrożenie aplikacji Odoo
- [ ] Utwórz plik `odoo-deployment.yaml` dla Odoo, używając obrazu Docker: `docker.io/bitnami/odoo:16`.
- [ ] W pliku Deployment skonfiguruj zmienne środowiskowe do połączenia z PostgreSQL (np. host, użytkownik, hasło).
- [ ] Zaaplikuj plik YAML do klastra:
  ```bash
  kubectl apply -f odoo-deployment.yaml -n odoo
  ```
- [ ] Zweryfikuj stan Poda:
  ```bash
  kubectl get pods -n odoo
  ```

## 5. Stworzenie obiektu Service dla Odoo
- [ ] Utwórz plik `odoo-service.yaml` z konfiguracją Service typu `ClusterIP`.
- [ ] Zaaplikuj plik YAML do klastra:
  ```bash
  kubectl apply -f odoo-service.yaml -n odoo
  ```

## 6. Testowanie aplikacji Odoo
- [ ] Użyj `kubectl port-forward`, aby uzyskać dostęp do aplikacji Odoo w przeglądarce:
  ```bash
  kubectl port-forward svc/<nazwa-service-dla-odoo> 8069:8069 -n odoo
  ```
- [ ] Otwórz przeglądarkę i wejdź na adres: [http://localhost:8069](http://localhost:8069).
- [ ] Zaloguj się do aplikacji Odoo, używając domyślnego emaila i hasła:
  - Email: `user@example.com`
  - Hasło: `bitnami`

## 7. Debugowanie
- [ ] Jeśli Pody się nie uruchamiają, sprawdź logi:
  ```bash
  kubectl logs <nazwa-poda> -n odoo
  ```
- [ ] Sprawdź status Podów i ich wydarzenia:
  ```bash
  kubectl describe pod <nazwa-poda> -n odoo
  ```

## 8. Dokumentacja
- [ ] Upewnij się, że pliki `Deployment` i `Service` są odpowiednio opisane i gotowe do udostępnienia w repozytorium.
