
# Udostępnienie aplikacji Odoo za pomocą Ingress na klastrze w chmurze

## **Wymagania wstępne**

Przed przystąpieniem do tego ćwiczenia upewnij się, że:

1. Aplikacja **Odoo** działa poprawnie na klastrze (patrz moduł 2).
2. Korzystasz z **klastra w chmurze** (np. GKE, AKS, EKS).
3. Masz zainstalowany **Ingress Controller**.
4. Usunąłeś wszelkie wcześniejsze obiekty **Ingress Resource**.

---

## **Kroki do wykonania**

1. **Weryfikacja istniejących zasobów Ingress**  
   Sprawdź, czy istnieją jakiekolwiek obiekty Ingress:
   ```bash
   kubectl get ingress -A
   ```
   Usuń wszystkie istniejące obiekty Ingress:
   ```bash
   kubectl delete ingress --all -A
   ```

2. **Sprawdzenie Ingress Controller**  
   Zweryfikuj, czy Ingress Controller jest zainstalowany:
   ```bash
   kubectl get all -n ingress-nginx
   ```
   Jeśli Ingress Controller nie jest zainstalowany, zainstaluj go zgodnie z dokumentacją:
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
   ```

3. **Stworzenie obiektu Ingress Resource**  
   Przygotuj plik `ingress-odoo.yaml` z konfiguracją Ingress Resource.

   Przykład konfiguracji:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: odoo-ingress
     namespace: odoo
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     ingressClassName: nginx
     rules:
     - host: odoo.example.com
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: odoo-service
               port:
                 number: 8069
   ```

4. **Wdrożenie obiektu Ingress**  
   Wdróż przygotowany plik:
   ```bash
   kubectl apply -f ingress-odoo.yaml
   ```

5. **Odczytanie publicznego adresu IP**  
   Sprawdź publiczny adres IP:
   ```bash
   kubectl get ingress -n odoo
   ```

6. **Testowanie aplikacji**  
   Otwórz przeglądarkę i przejdź na publiczny adres IP lub skonfigurowany host (`odoo.example.com`).

---

## **Weryfikacja poprawności**

- Jeśli aplikacja **Odoo** ładuje się w przeglądarce pod publicznym adresem IP, zadanie zostało wykonane poprawnie.
- W przypadku problemów, sprawdź plik **TODO.md** z podpowiedziami.

---

## **Pliki towarzyszące**

- `ingress-odoo.yaml` – Konfiguracja Ingress Resource.
- `TODO.md` – Wskazówki rozwiązywania problemów.
