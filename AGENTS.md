# Instrukcje pracy nad Home Assistantem

## Komunikacja

- Komunikuj się z użytkownikiem po polsku.
- Teksty interfejsu kierowane do użytkowników pisz neutralnym, formalnym i zrozumiałym językiem. Zachowuj zwięzłość, unikaj potocznych określeń i zbędnych powtórzeń.

## Projekt i zależności

- Repozytorium zawiera konfigurację Home Assistant Container i Docker Compose. Zależności Pythona są częścią oficjalnego obrazu Home Assistanta.
- Przy aktualizacji sprawdź bieżące stabilne wydanie w oficjalnym `home-assistant/version/stable.json`, ustaw konkretny tag obrazu i zweryfikuj konfigurację narzędziem z nowego obrazu.
- Zachowuj wersje bibliotek dobrane przez Home Assistant. Nie aktualizuj ich osobno przez `pip` w gotowym obrazie.
- Stary własny dodatek Shelly zastąpiono wbudowaną integracją REST. Konfiguracja czterech urządzeń znajduje się w `config/switches.yaml`.
- Zachowuj nazwy i adresy urządzeń, chyba że użytkownik poda zmianę lub odczyt z sieci potwierdzi konieczność korekty. Dwa powtarzające się klucze starej konfiguracji zostały rozdzielone na osobne urządzenia.

## Kontynuacja na Hydropolis

- Docelowa instalacja: serwer `hydropolis`, konto `hydropolis`, katalog `/home/hydropolis/homeassistant`.
- Użytkownik zdecydował, że sam sklonuje repozytorium i rozpocznie dalszą sesję na serwerze. Przygotowanie zmian w lokalnym repozytorium nie uruchamia wdrożenia na serwerze.
- Po zleceniu wdrożenia sprawdź bieżący stan katalogu, kontenerów, dostępne porty i instrukcję w `README.md`. Jeśli instalacja już działa, wykonaj kopię danych przed jej aktualizacją.
- Stosuj projekt Compose `homeassistant`. Panel korzysta z portu `9107`, a kontener z sieci hosta. Na AlmaLinux z SELinux zachowaj opcję `:Z` dla katalogu konfiguracji.
- Na serwerze działają także inne projekty. Polecenia Dockera ograniczaj do Home Assistanta; nie zatrzymuj innych usług ani nie uruchamiaj globalnego czyszczenia zasobów Dockera.
- Stan aplikacji, konta, tokeny i baza znajdują się w `config/.storage` oraz bazie SQLite. Zachowaj te dane i nie dodawaj ich do Git.
- Urządzenia `192.168.1.130`–`192.168.1.133` nie odpowiadały przez HTTP podczas sprawdzenia 2 października 2026. Ponów odczyt ich dostępności przed oceną integracji. Nie przełączaj oświetlenia w ramach testów.
- Pierwsze konto administratora Home Assistanta użytkownik tworzy przez formularz w panelu.

## Weryfikacja zmian

Uruchom `git diff --check`, `docker compose config --quiet` oraz sprawdzenie konfiguracji Home Assistanta opisane w `README.md`. Przy wdrożeniu sprawdź dodatkowo stan zdrowia kontenera, odpowiedź HTTP panelu i logi aplikacji. Oddziel błędy konfiguracji od braku połączenia z urządzeniami LAN.
