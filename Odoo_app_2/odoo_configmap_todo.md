# TODO: Zmiana konfiguracji Odoo i Postgres

## Zadania do wykonania:

### Krok 1: Stwórz obiekt ConfigMap
- [x] Utwórz ConfigMap o nazwie `odoo-configmap`.
- [x] Dodaj właściwość: `DB_USER: odoo`.

### Krok 2: Ustaw nazwę użytkownika z ConfigMap
- [x] Skonfiguruj Deployment **Odoo**, aby korzystał z `DB_USER` wczytanego z ConfigMap.
- [x] Skonfiguruj Deployment **Postgres**, aby korzystał z `DB_USER` wczytanego z ConfigMap.

### Krok 3: Stwórz obiekt Secret
- [x] Utwórz Secret o nazwie `odoo-secret`.
- [x] Dodaj właściwość: `DB_PASSWORD: k8smaestro` (zakodowaną w base64).

### Krok 4: Ustaw hasło z Secret
- [x] Skonfiguruj Deployment **Odoo**, aby korzystał z `DB_PASSWORD` wczytanego z Secret.
- [x] Skonfiguruj Deployment **Postgres**, aby korzystał z `DB_PASSWORD` wczytanego z Secret.

### Krok 5: Wdróż zmiany
- [x] Wdróż wszystkie zmodyfikowane obiekty w klastrze Kubernetes.

### Krok 6: Zweryfikuj poprawność działania
- [x] Uruchom `kubectl port-forward` dla aplikacji Odoo.
- [x] Sprawdź w przeglądarce, czy pojawia się ekran logowania do Odoo.

---

### Wskazówki
- Użyj przykładowych plików `basic-pod` lub `mysql-secret.yml` jako wzoru.
- Zakoduj wartości Secret w base64:
  ```bash
  echo -n "wartość" | base64
  ```
