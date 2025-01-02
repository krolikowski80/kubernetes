
# Analiza pliku manifestu Kubernetes

Poniższy dokument zawiera szczegółową analizę manifestu Kubernetes linia po linii. Zawiera wszystkie ważne informacje, które mogą być przydatne podczas tworzenia własnych manifestów.

---

```yaml
apiVersion: apps/v1
```

### Wyjaśnienie:
- **apiVersion**: Określa wersję API Kubernetes, która ma być użyta do zdefiniowania zasobu. 
- W tym przypadku **apps/v1** wskazuje na standardowe API dla obiektów takich jak Deployment.

---

```yaml
kind: Deployment
```

### Wyjaśnienie:
- **kind**: Określa rodzaj zasobu Kubernetes. 
- **Deployment** definiuje zarządzany zestaw replik Podów, umożliwiając ich automatyczne skalowanie, aktualizacje i odtwarzanie w razie awarii.

---

```yaml
metadata:
  name: payment-deployment
  namespace: default
```

### Wyjaśnienie:
- **metadata**: Sekcja zawiera dane opisowe zasobu.
  - **name**: Unikalna nazwa Deploymentu w ramach wybranego namespace.
  - **namespace**: Określa przestrzeń nazw, w której zasób jest tworzony. Domyślnie jest to `default`.

---

```yaml
spec:
  replicas: 1
```

### Wyjaśnienie:
- **spec**: Sekcja zawierająca specyfikację zasobu.
  - **replicas**: Liczba replik Podów, które mają zostać utworzone. Tutaj ustawiona na `1`, co oznacza jedną instancję aplikacji.

---

```yaml
  selector:
    matchLabels:
      app: payment-app
```

### Wyjaśnienie:
- **selector**: Określa kryteria wyboru dla Podów zarządzanych przez ten Deployment.
  - **matchLabels**: Używany do dopasowywania etykiet Podów. Tylko Pody z etykietą `app: payment-app` będą zarządzane.

---

```yaml
  template:
    metadata:
      labels:
        app: payment-app
```

### Wyjaśnienie:
- **template**: Określa szablon dla Podów, które mają być utworzone.
  - **metadata**: Sekcja etykiet szablonu Poda.
    - **labels**: Definiuje etykiety dla Podów tworzonych z tego szablonu. 

Etykieta `app: payment-app` jest kluczowa, ponieważ musi odpowiadać selectorowi Deploymentu.

---

```yaml
    spec:
      containers:
        - name: busybox-payment
          image: busybox:1.36.0
```

### Wyjaśnienie:
- **containers**: Lista kontenerów, które będą uruchamiane w każdym Podzie.
  - **name**: Nazwa kontenera, tutaj `busybox-payment`.
  - **image**: Obraz Dockerowy, który ma być użyty. W tym przypadku `busybox:1.36.0`.

---

```yaml
          command:
            - sh
            - -c
            - |
              echo "MERCHANT_ID: $MERCHANT_ID"
              echo "PAYMENT_GATEWAY_API_KEY: $PAYMENT_GATEWAY_API_KEY"
```

### Wyjaśnienie:
- **command**: Definiuje polecenie uruchamiane przez kontener po jego starcie.
  - W tym przypadku uruchamiana jest powłoka `sh` z flagą `-c`, aby wykonać zestaw poleceń:
    - Wyświetlenie zmiennych środowiskowych `MERCHANT_ID` i `PAYMENT_GATEWAY_API_KEY`.

---

```yaml
          env:
            - name: MERCHANT_ID
              valueFrom:
                configMapKeyRef:
                  name: payment-config
                  key: MERCHANT_ID
```

### Wyjaśnienie:
- **env**: Lista zmiennych środowiskowych przekazywanych do kontenera.
  - **name**: Nazwa zmiennej, tutaj `MERCHANT_ID`.
  - **valueFrom**: Pobiera wartość z innego zasobu.
    - **configMapKeyRef**: Odwołanie do klucza w ConfigMap.
      - **name**: Nazwa ConfigMap, tutaj `payment-config`.
      - **key**: Klucz w ConfigMap, tutaj `MERCHANT_ID`.

---

```yaml
            - name: PAYMENT_GATEWAY_API_KEY
              valueFrom:
                secretKeyRef:
                  name: payment-secret
                  key: PAYMENT_GATEWAY_API_KEY
```

### Wyjaśnienie:
- **name**: Nazwa zmiennej, tutaj `PAYMENT_GATEWAY_API_KEY`.
- **valueFrom**: Pobiera wartość z Secret.
  - **secretKeyRef**: Odwołanie do klucza w Secret.
    - **name**: Nazwa Secret, tutaj `payment-secret`.
    - **key**: Klucz w Secret, tutaj `PAYMENT_GATEWAY_API_KEY`.

---

# Wnioski i dobre praktyki
1. **Użycie ConfigMap i Secret**:
   - ConfigMap przechowuje dane niestrukturalne, takie jak konfiguracja aplikacji.
   - Secret przechowuje dane wrażliwe (np. hasła, klucze API) w formie zaszyfrowanej.

2. **Etykiety i selektory**:
   - Kluczowe dla prawidłowego dopasowania Deploymentu do Podów.

3. **Szablon Poda**:
   - Wszystkie elementy szablonu (metadata, spec) muszą być precyzyjnie zdefiniowane, aby Deployment działał poprawnie.

4. **Dekompozycja na pliki YAML**:
   - W większych projektach warto oddzielić Deployment, ConfigMap i Secret do osobnych plików dla lepszej organizacji.

Gotowe do wykorzystania jako źródło wiedzy podczas tworzenia własnych manifestów.
