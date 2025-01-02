
# TODO: Trwałość danych dla aplikacji Odoo (GCP)

## Zadania do wykonania

1. **Przygotowanie klastra GKE:**
   - [ ] Zaloguj się do Google Cloud Console.
   - [ ] Upewnij się, że projekt i API GKE są aktywowane.
   - [ ] Stwórz klaster Kubernetes (GKE).
   - [ ] Skonfiguruj `kubectl` do pracy z klastrem.

2. **Usunięcie istniejących zasobów Odoo:**
   - [ ] Usuń wszystkie istniejące zasoby Odoo z klastra.

3. **Persistent Volume Claim (PVC):**
   - [ ] Sprawdź dostępne StorageClassy (`kubectl get storageclass`).
   - [ ] Stwórz plik `odoo-pvc.yaml` z konfiguracją PVC.
   - [ ] Wdróż PVC (`kubectl apply -f odoo-pvc.yaml`).

4. **Deployment aplikacji Odoo:**
   - [ ] Zaktualizuj plik Deploymentu:
     - [ ] Dodaj sekcję `volumes` dla PVC.
     - [ ] Dodaj sekcję `volumeMounts` dla ścieżki `/bitnami/odoo`.
   - [ ] Wdróż Deployment (`kubectl apply -f <plik-deployment-odoo>`).

5. **Weryfikacja:**
   - [ ] Sprawdź, czy PVC zostało poprawnie podmontowane (`kubectl describe pod <nazwa-poda>`).
   - [ ] Upewnij się, że aplikacja Odoo działa poprawnie.

6. **(Opcjonalnie):**
   - [ ] Zaloguj się do Google Cloud Console i sprawdź szczegóły utworzonego storage.
