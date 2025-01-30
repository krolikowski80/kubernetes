# Konfiguracja Deployment dla Odoo z Rolling Update

## Wymagania wstępne

Zanim przystąpisz do tego ćwiczenia, upewnij się, że aplikacja **Odoo** działa poprawnie. Jeśli nie, wróć do modułu 2 i skonfiguruj ją poprawnie.

## Zadanie

Twoim zadaniem jest rozszerzenie definicji obiektu **Deployment** dla aplikacji Odoo poprzez:

1. Dodanie limitów (`limits`) i żądań (`requests`) dla zasobów kontenera.
2. Skonfigurowanie mechanizmu **Rolling Update** tak, aby stare Pody były usuwane dopiero wtedy, gdy nowy Pod będzie w pełni gotowy do obsłużenia ruchu użytkowników.

Po wykonaniu tej części przejdź do kolejnego zadania.

---

## Część 1: Modyfikacja Deployment

Otwórz plik `odoo-deployment.yaml` i dodaj poniższą konfigurację do sekcji `spec.containers.resources`:

```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "250m"
  limits:
    memory: "1Gi"
    cpu: "500m"
```

Następnie w sekcji `strategy` dodaj strategię **Rolling Update**:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

> **Wyjaśnienie:** 
> - `maxUnavailable: 0` – zapewnia, że żaden Pod nie zostanie usunięty, dopóki nowy nie będzie gotowy.
> - `maxSurge: 1` – umożliwia uruchomienie jednego dodatkowego Poda w trakcie aktualizacji.

Zaaplikuj zmiany na klaster:
```bash
kubectl apply -f odoo-deployment.yaml
```

---

## Część 2: Zmiana obrazu aplikacji i test Rolling Update

1. Zmień obraz w `odoo-deployment.yaml` na nową wersję:

```yaml
image: docker.io/bitnami/odoo:15
```

2. Zastosuj zmiany na klastrze:
```bash
kubectl apply -f odoo-deployment.yaml
```

3. Sprawdź status wdrożenia:
```bash
kubectl rollout status deployment odoo
```

4. Zweryfikuj, czy nowe Pody działają poprawnie:
```bash
kubectl get pods
```

---

## Wskazówki

- Spróbuj samodzielnie dobrać odpowiednie wartości `requests` i `limits`, korzystając z dokumentacji Odoo.
- Zbyt niski limit pamięci może powodować problemy z uruchomieniem aplikacji.
- Monitoruj proces wdrożenia za pomocą `kubectl get pods --watch`.

---

## Podsumowanie

Po wykonaniu tych kroków, aplikacja Odoo powinna działać z nową strategią Rolling Update oraz poprawnie ustawionymi zasobami. 🎯

Jeśli masz problem, spróbuj przeanalizować logi (`kubectl logs pod-name`) lub skorzystaj z dokumentacji Kubernetes i Odoo.
