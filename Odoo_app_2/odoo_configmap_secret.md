# Zadanie: Zmiana konfiguracji Odoo i Postgres z wykorzystaniem ConfigMap i Secret

## Wprowadzenie

### Ważne:
Aby przystąpić do tego ćwiczenia, aplikacja Odoo musi działać poprawnie (patrz moduł 2).

---

## Zadanie
Twoim zadaniem będzie zmiana zachowania aplikacji Odoo i Postgres, tak by:

- **Nazwa użytkownika bazy danych** była wczytywana z obiektu **ConfigMap** (poprzez zmienne środowiskowe), zamiast bezpośrednio w każdym z Deploymentów jako plain-text.
- **Hasło do bazy danych** było wczytywane z obiektu **Secret**.

### Dlaczego ConfigMap?
Jeśli mielibyśmy 10 mikroserwisów aplikacji Odoo, i każdy z nich łączyłby się do bazy danych, to nazwę użytkownika będziemy mieli w jednym miejscu (w ConfigMap). Gdy kiedykolwiek potrzebowalibyśmy zmienić nazwę użytkownika do bazy, zrobimy to w jednym miejscu, zamiast w 11-stu miejscach.

---

## Krok po kroku

### Krok 1: Stwórz obiekt ConfigMap
1. Utwórz obiekt **ConfigMap** o nazwie `odoo-configmap`.
2. Dodaj w nim właściwość:
   ```yaml
   DB_USER: odoo
   ```

### Krok 2: Ustaw nazwę użytkownika z ConfigMap
1. Spraw, by nazwa użytkownika bazy danych była wczytywana z obiektu ConfigMap dla:
   - Deploymentu **Odoo**
   - Deploymentu **Postgres**

### Krok 3: Stwórz obiekt Secret
1. Utwórz obiekt **Secret** o nazwie `odoo-secret`.
2. Dodaj w nim właściwość:
   ```yaml
   DB_PASSWORD: k8smaestro
   ```

### Krok 4: Ustaw hasło z Secret
1. Spraw, by hasło do bazy danych było wczytywane z obiektu Secret dla:
   - Deploymentu **Odoo**
   - Deploymentu **Postgres**

### Krok 5: Wdróż zmiany
1. Wdróż wszystkie zmodyfikowane obiekty w klastrze Kubernetes.
2. Zaobserwuj zmiany.

### Krok 6: Zweryfikuj poprawność działania
1. Skorzystaj z `kubectl port-forward`, aby podłączyć się do aplikacji Odoo z przeglądarki (patrz moduł drugi).
2. Zweryfikuj, czy po przejściu do przeglądarki pojawia się ekran logowania do aplikacji Odoo.

---

## Wskazówki

1. Jeśli napotkasz problemy ze składnią, zerknij na przykłady:
   - `basic-pod`
   - `mysql-secret.yml`

2. Możesz podzielić zadanie na dwa kroki:
   - **Krok 1**: Wykonaj punkty 1, 2, 5, 6 i zweryfikuj zmiany.
   - **Krok 2**: Wykonaj punkty 3, 4, 5, 6.

3. Pamiętaj, że wartości **Secret** muszą być przechowywane w formacie **base64**.
   Możesz je zakodować, używając polecenia:
   ```bash
   echo -n "wartość" | base64
   ```
