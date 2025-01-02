
# Usuwanie klastra GKE i konfiguracji lokalnej

## Krok 1: Usuń klaster GKE

1. **Zaloguj się do Google Cloud CLI (jeśli jeszcze nie jesteś zalogowany):**
   ```bash
   gcloud auth login
   ```

2. **Wybierz projekt, w którym znajduje się Twój klaster:**
   ```bash
   gcloud config set project [PROJECT_ID]
   ```

3. **Wyświetl listę istniejących klastrów, aby upewnić się, który chcesz usunąć:**
   ```bash
   gcloud container clusters list
   ```

4. **Usuń wybrany klaster GKE:**
   ```bash
   gcloud container clusters delete [CLUSTER_NAME] --zone [ZONE]
   ```
   - Zastąp `[CLUSTER_NAME]` nazwą swojego klastra.
   - Zastąp `[ZONE]` strefą, w której znajduje się Twój klaster.

   Przykład:
   ```bash
   gcloud container clusters delete my-cluster --zone us-central1-a
   ```

## Krok 2: Usuń konfigurację lokalną z `kubectl`

1. **Usuń wpis klastra z pliku kubeconfig:**
   ```bash
   kubectl config delete-context [CONTEXT_NAME]
   ```
   - Aby zobaczyć listę kontekstów, użyj:
     ```bash
     kubectl config get-contexts
     ```

2. **Usuń lokalny plik kubeconfig (opcjonalnie):**
   Jeśli chcesz usunąć cały plik konfiguracyjny `kubeconfig`, wykonaj:
   ```bash
   rm ~/.kube/config
   ```

## Krok 3: Zweryfikuj usunięcie

1. **Sprawdź, czy klaster został usunięty:**
   ```bash
   gcloud container clusters list
   ```

2. **Sprawdź, czy lokalna konfiguracja została wyczyszczona:**
   ```bash
   kubectl config get-contexts
   ```

---

## Podsumowanie

Po wykonaniu powyższych kroków:
- Klaster GKE zostanie usunięty.
- Konfiguracja lokalna z `kubectl` zostanie wyczyszczona.

