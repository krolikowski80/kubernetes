
# TODO: Wdrożenie aplikacji Shipy w Kubernetes

## Cel
Nauczyć się tworzenia i wdrażania obiektów Kubernetes takich jak **Deployment**, **Service**, oraz **Ingress** na przykładzie aplikacji Shipy.

## Zadania do wykonania

1. **Deployment**
   - Stwórz plik YAML opisujący Deployment dla aplikacji Shipy.
   - Użyj obrazu `k8smaestro/shipy-webapp:1.41.5`.
   - Aplikacja działa na porcie `7000`, więc upewnij się, że kontenerPort jest ustawiony poprawnie.
   - Ustaw jeden replikat (replica).

2. **Service**
   - Utwórz plik YAML opisujący Service dla aplikacji.
   - Typ Service: `ClusterIP`.
   - Połącz Service z Deployment za pomocą odpowiednich etykiet.

3. **Ingress**
   - Sprawdź, czy Ingress Controller (np. nginx) jest zainstalowany na Twoim klastrze.
   - Zdefiniuj plik YAML dla obiektu Ingress, który wystawi aplikację na zewnątrz.
   - Ustaw poprawną nazwę klasy Ingress (np. `nginx`) i nazwę Service z kroku 2.

4. **Wdrażanie**
   - Wdroż wszystkie pliki YAML do klastra Kubernetes za pomocą `kubectl apply`.
   - Użyj namespace `shipy` dla wszystkich obiektów.

5. **Weryfikacja**
   - Pobierz publiczny adres IP Ingress Controller za pomocą `kubectl get ingress`.
   - Przejdź w przeglądarce pod uzyskany adres, aby upewnić się, że aplikacja działa.

6. **Debugowanie**
   - Jeśli aplikacja nie działa:
     - Użyj `kubectl port-forward` na obiekcie Service, aby przekierować ruch na localhost.
     - Sprawdź logi pod kątem błędów w konfiguracji.

---

## Wskazówki
- Używaj `kubectl explain`, aby zrozumieć strukturę obiektów YAML.
- Sprawdź poprzednie lekcje dotyczące Ingress Controller i Deployment.
- Skoncentruj się na powiązaniach między Deployment, Service, i Ingress.

Powodzenia!
