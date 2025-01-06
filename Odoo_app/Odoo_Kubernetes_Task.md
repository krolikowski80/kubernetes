
# Wdrożenie systemu Odoo

Odoo to kompleksowe oprogramowanie do zarządzania przedsiębiorstwem, które zawiera wiele modułów, takich jak CRM, księgowość, sprzedaż, zakupy, magazynowanie i wiele innych. 
Odoo może być wdrożone jako samodzielna aplikacja lub jako platforma do tworzenia dedykowanych aplikacji biznesowych. 
Więcej szczegółów znajdziesz [tutaj](https://www.odoo.com).

## Zanim zaczniesz

Jak sama nazwa wskazuje, jest to ćwiczenie do samodzielnego wykonania. Oczekujemy od Ciebie samodzielności, korzystania z Google (i ChatGPT) i … cierpliwości. 
Jeżeli zadanie jest dla Ciebie zbyt trudne, możesz je pominąć i kontynuować szkolenie. Możesz też do niego wrócić po przerobieniu kolejnych modułów. 
Wykonanie zadanie nie jest wymagane, aby przejść przez kolejne moduły szkolenia.

### Nie podsyłaj gotowego rozwiązania na Discord

Jeśli udało Ci się wykonać zadanie, gratulujemy. Możesz pochwalić się zrzutem ekranu na Discord z działającego wdrożenia, ale prosimy – NIE wysyłaj gotowych skryptów na Discord, bo popsuło by to "zabawę" innym uczestnikom szkolenia.



## Twoje zadanie

Twoim zadaniem jest wdrożenie bazy danych Postgres oraz aplikacji Odoo na klastrze Kubernetes, używając (osobnych) obiektów Deployment oraz obiektów Service w namespace o nazwie `odoo`.

1. Aby ułatwić Ci zadanie, przygotowaliśmy już definicję obiektu Deployment dla Postgresa: 
   [postgres-deployment.yaml](https://raw.githubusercontent.com/dnaprawa/KubernetesMaestro/main/02/self-study/postgres-deployment.yaml). 
   Skopiuj go i ustaw namespace na `odoo`.
   
2. Stwórz pliki YAML zawierający definicje obiektów Deployment. Użyj jednej repliki (1 Pod). 
   Do wdrożenia aplikacji Odoo, użyj gotowego obrazu dockerowego: [docker.io/bitnami/odoo:16](https://hub.docker.com/r/bitnami/odoo/). 
   Znajdziesz tam instrukcję – w jaki sposób uruchomić aplikację za pomocą Dockera i wykorzystać tę wiedzę do uruchomienia w Kubernetes. 
   Ten link również może Ci się przydać: [docker-compose.yml](https://raw.githubusercontent.com/bitnami/containers/main/bitnami/odoo/docker-compose.yml).

3. Dla każdego obiektu Deployment stwórz również obiekt Service, który umożliwi dostęp sieciowy do Podów wewnątrz klastra Kubernetes (Service typu ClusterIP).

4. Skonfiguruj Service, aby przekierowywał ruch na odpowiedni port kontenera (wystarczy jedna replika / Pod, stąd określenie "kontenera"). 
   Zwróć uwagę na nazwę obiektu Service dla Postgresa. Będzie Ci to potrzebne w obiekcie Deployment dla aplikacji Odoo.

5. Jeśli pojawią się jakieś wyzwania – wróć do poprzednich lekcji i spróbuj przez analogię samodzielnie rozwiązać problem – 
   na przykład mechanizm komunikacji pomiędzy dwoma serwisami w Kubernetes.

6. Zwróć uwagę, w jaki sposób w aplikacji Odoo przekazywane jest połączenie do bazy danych. Dla ułatwienia – są to zmienne środowiskowe. 
   Ich nazwy znajdziesz w/w linkach. Pomiń kwestie volumenów. Jako że jest to moduł drugi szkolenia, 
   nie będziemy wykorzystywać jeszcze wszystkich najlepszych praktyk Kubernetes. Celem jest uruchomienie (samodzielnie) pierwszej aplikacji.

7. Skup się na tym, by aplikacja uruchomiła się na Twoim klastrze, i by móc z poziomu własnej przeglądarki “po niej poklikać”.

8. Wykorzystaj narzędzie `kubectl apply`, aby wdrożyć aplikację na klastrze Kubernetes. Uruchom aplikację w przeglądarce.

### Kompletne rozwiązanie zadanie powinno zawierać:

- pliki YAML z definicją Deployment i Service dla bazy danych oraz dla aplikacji Odoo,
- wdrożenie w Kubernetes,
- uruchomienie Odoo z poziomu Twojej przeglądarki (wykorzystaj `kubectl port-forward`). Nie używaj `kubectl proxy` (są z tym problemy),

---

## Logowanie do Odoo

Według dokumentacji możemy tym sterować przez zmienne środowiskowe:

- **ODOO_EMAIL**: Odoo application email. Default: `user@example.com`
- **ODOO_PASSWORD**: Odoo application password. Default: `bitnami`

---

## Dodatkowa pomoc

Jeśli pojawiłyby się jakieś problemy z uruchomieniem – np. `CrashLoopBackoff` – zajrzyj do [TEJ](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/) 
i/lub [TEJ](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/) lekcji. Są to lekcje o debugowaniu Podów.

---

## Opis poszczególnych elementów definicji obiektu Deployment dla Postgresa:

- **apiVersion**: wersja API, w tym przypadku `apps/v1`
- **kind**: rodzaj obiektu, w tym przypadku `Deployment`
- **metadata**: metadane, takie jak nazwa wdrożenia
- **spec**: definicja specyfikacji wdrożenia, w tym liczba replik i selektor etykiet
- **template**: definicja szablonu dla podów tworzonych przez wdrożenie
- **containers**: definicja kontenera, w tym nazwa, obraz, porty i zmienne środowiskowe
- **env**: lista zmiennych środowiskowych, w tym hasło do bazy danych
- **volumeMounts**: montowanie woluminu dla przechowywania danych bazy PostgreSQL
- **volumes**: definicja woluminu, w tym typ i nazwa woluminu.

Ten przykład definiuje wdrożenie PostgreSQL z jednym podem i jednym kontenerem, używając obrazu `bitnami/postgresql:15`. 
Hasło do bazy danych jest ustawiane manualnie i jawnie. Zapamiętaj, że NIE jest to dobra praktyka, 
ale nie chcemy komplikować ćwiczenia na tym etapie. W kolejnych modułach do tego wrócimy.
