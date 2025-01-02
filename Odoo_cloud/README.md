
# Zadanie: Trwałość danych dla aplikacji Odoo w Kubernetes (GCP)

## Opis zadania

Twoim zadaniem będzie rozszerzenie funkcjonalności aplikacji Odoo poprzez zapewnienie trwałości danych dla plików aplikacji zlokalizowanych w kontenerze w ścieżce `/bitnami/odoo`. W tym celu należy wykorzystać mechanizm StorageClass do automatycznego zarządzania Persistent Volumes (PV) i Persistent Volume Claim (PVC).

## Wymagania wstępne

1. **Klaster Kubernetes w GCP**: 
   - Stworzony za pomocą Google Kubernetes Engine (GKE).
   - Narzędzie `kubectl` skonfigurowane do pracy z Twoim klastrem.

2. **Aplikacja Odoo**: 
   - Poprawnie wdrożona w module 2.
   - Przygotowanie do wdrożenia od zera.

3. **Dostęp do Google Cloud Console**:
   - Uprawnienia do tworzenia zasobów w GCP.

## Instrukcja krok po kroku

### 1. Przygotowanie klastra Kubernetes w GCP

1. Zaloguj się do Google Cloud Console: [https://console.cloud.google.com](https://console.cloud.google.com).
2. Upewnij się, że masz aktywowany projekt i API Google Kubernetes Engine (GKE).
3. Stwórz klaster GKE:
   - Przejdź do sekcji **Kubernetes Engine** > **Clusters**.
   - Kliknij **Create Cluster** i skonfiguruj klaster zgodnie z wymaganiami (np. liczba węzłów, region).
4. Skonfiguruj `kubectl` do pracy z klastrem:
   ```bash
   gcloud container clusters get-credentials <nazwa-klastra> --zone <region> --project <projekt>
   ```

### 2. Usuń istniejące zasoby Odoo

Jeśli masz już wdrożoną aplikację Odoo, usuń wszystkie powiązane zasoby:
```bash
kubectl delete -f <plik-konfiguracji-odoo>
```

### 3. Sprawdź dostępne StorageClassy

Sprawdź, jakie StorageClassy są dostępne w Twoim klastrze:
```bash
kubectl get storageclass
```

W GCP zazwyczaj dostępne są klasy, takie jak `standard` (domyślna) lub `premium-rwo`. Wybierz odpowiednią klasę.

### 4. Stwórz Persistent Volume Claim (PVC)

1. Utwórz plik `odoo-pvc.yaml` z następującą konfiguracją:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: odoo-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 10Gi
     storageClassName: <nazwa-wybranej-klasy>
   ```
2. Wdróż PVC na klastrze:
   ```bash
   kubectl apply -f odoo-pvc.yaml
   ```

### 5. Zaktualizuj Deployment aplikacji Odoo

1. W pliku Deploymentu aplikacji Odoo dodaj sekcję `volumes`:
   ```yaml
   volumes:
     - name: odoo-data
       persistentVolumeClaim:
         claimName: odoo-pvc
   ```
2. Zaktualizuj sekcję `volumeMounts` w kontenerze Odoo:
   ```yaml
   volumeMounts:
     - mountPath: /bitnami/odoo
       name: odoo-data
   ```
3. Wdróż Deployment:
   ```bash
   kubectl apply -f <plik-deployment-odoo>
   ```

### 6. Weryfikacja

1. Sprawdź, czy PVC zostało poprawnie podmontowane:
   ```bash
   kubectl describe pod <nazwa-poda>
   ```
2. Sprawdź, czy aplikacja działa poprawnie.

### 7. (Opcjonalnie) Podejrzyj storage w konsoli GCP

1. Zaloguj się do Google Cloud Console.
2. Przejdź do sekcji **Filestore** lub **Persistent Disks**, aby zobaczyć utworzony storage.
