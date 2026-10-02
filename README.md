# Home Assistant na Hydropolis

Repozytorium zawiera konfigurację domowego Home Assistanta uruchamianego przez Docker Compose w katalogu `/home/hydropolis/homeassistant`, na koncie `hydropolis`. Najpierw należy sklonować repozytorium na serwer; wdrożenie można kontynuować podczas kolejnej sesji.

Obraz `ghcr.io/home-assistant/home-assistant:2026.9.4` odpowiada najnowszemu stabilnemu wydaniu sprawdzonemu 2 października 2026. Wersję potwierdzają [wykaz stabilnych wersji](https://github.com/home-assistant/version/blob/master/stable.json) i [wydanie 2026.9.4](https://github.com/home-assistant/core/releases/tag/2026.9.4). Biblioteki Pythona dostarcza oficjalny obraz, w wersjach dobranych przez Home Assistant. Aktualizacja obrazu aktualizuje również zależności aplikacji. Repozytorium nie wymaga osobnej instalacji bibliotek przez `pip`.

## Sklonowanie repozytorium

Na komputerze z dostępem do serwera:

```bash
ssh hydropolis@hydropolis
git clone https://github.com/mszpiler/homeassistant.git /home/hydropolis/homeassistant
cd /home/hydropolis/homeassistant
```

Otwórz sesję agenta w katalogu `/home/hydropolis/homeassistant` i poproś o kontynuowanie wdrożenia zgodnie z `AGENTS.md` oraz instrukcją poniżej.

## Sprawdzenie i uruchomienie

Polecenia wykonuj na koncie `hydropolis`, w katalogu repozytorium. Docker Engine i Docker Compose są już zainstalowane na serwerze. Konto ma dostęp do Dockera.

```bash
docker compose config --quiet
docker compose pull
docker compose run --rm --no-deps --entrypoint python3 homeassistant \
  -m homeassistant --script check_config --config /config
```

Po pomyślnej walidacji uruchom usługę i sprawdź jej stan:

```bash
docker compose up -d --wait --wait-timeout 300
docker compose ps
docker compose logs --tail=100 homeassistant
```

Po uruchomieniu otwórz jeden z adresów:

- sieć domowa: <http://192.168.1.250:9107>;
- Tailscale: <http://hydropolis:9107> lub <http://100.107.43.56:9107>.

Pierwsze uruchomienie wyświetli formularz utworzenia konta administratora Home Assistanta. Zgodnie z [instrukcją Home Assistant Container](https://www.home-assistant.io/installation/linux/) kontener korzysta z sieci hosta, aby umożliwić wykrywanie urządzeń domowych. Port `9107` ustawiono w `config/configuration.yaml`; mapowanie portów Dockera nie jest potrzebne. Katalog konfiguracji ma opcję montowania `:Z`, wymaganą przez SELinux na Hydropolis.

Jeżeli panel odpowiada na `http://127.0.0.1:9107/` na serwerze, ale jest niedostępny przez LAN lub Tailscale, sprawdź aktywną strefę i reguły Firewalld. Zmiana reguł wymaga dostępu administratora systemu.

## Urządzenia Shelly

Plik `config/switches.yaml` zawiera cztery przełączniki obsługiwane przez wbudowaną integrację REST. Konfiguracja zachowuje API `/relay/0`, nazwy i adresy urządzeń pierwszej generacji:

| Urządzenie | Adres |
| --- | --- |
| Lampa nad stołem | `192.168.1.130` |
| Schody | `192.168.1.131` |
| Korytarz parter | `192.168.1.132` |
| Kuchnia wyspa | `192.168.1.133` |

W starej konfiguracji dwa urządzenia miały klucz `shelly_three`. Obecna lista ma cztery osobne identyfikatory. Usunięto własny dodatek korzystający z nieaktualnego API Home Assistanta. Stan jest odczytywany co 5 sekund; polecenia sterowania używają żądania POST z parametrem `turn=on` lub `turn=off`, zgodnie z [integracją REST](https://www.home-assistant.io/integrations/switch.rest/) i [API Shelly](https://shelly-api-docs.shelly.cloud/gen1/).

Podczas sprawdzenia 2 października 2026 żaden z adresów nie odpowiedział na żądanie HTTP z serwera. Po wdrożeniu zweryfikuj adresy, dostępność urządzeń i generację Shelly. Niedostępne urządzenia mogą opóźnić dodanie przełączników, a integracja ponawia próby połączenia.

Po potwierdzeniu urządzeń można przejść na natywną integrację Shelly przez **Ustawienia → Urządzenia i usługi → Dodaj integrację → Shelly**. Usuń odpowiedni przełącznik REST przy przejściu na natywną integrację, aby uniknąć podwójnych encji.

## Konfiguracja i dane

`config/configuration.yaml` zawiera położenie domu, strefę `Europe/Warsaw`, system metryczny, port panelu i domyślne integracje Home Assistanta. Obsługę Google TTS zachowano pod poprawną nazwą `google_translate`, z językiem polskim. Automatyzacje, skrypty i sceny mają osobne pliki YAML, dzięki czemu można nimi zarządzać z panelu.

Katalog `config` przechowuje też stan aplikacji, konta i bazę SQLite. Te dane oraz `secrets.yaml` są pomijane przez Git. Katalog należy zachować pomiędzy odtworzeniami kontenera.

## Kolejne aktualizacje

Przed aktualizacją działającej instalacji wykonaj kopię całego katalogu `config` przy zatrzymanym kontenerze. Konto `hydropolis` może odczytać pliki utworzone przez kontener za pomocą pomocniczego kontenera z tym samym obrazem:

```bash
cd /home/hydropolis/homeassistant
mkdir -p /home/hydropolis/homeassistant-backups
chmod 700 /home/hydropolis/homeassistant-backups
docker compose stop homeassistant
docker compose run --rm --no-deps --entrypoint tar \
  -v /home/hydropolis/homeassistant-backups:/backup:Z homeassistant \
  -czf "/backup/config-$(date +%Y%m%d-%H%M%S).tar.gz" -C /config .
docker compose start homeassistant
```

Następnie pobierz zmiany przez `git pull --ff-only`, sprawdź różnice w konfiguracji, pobierz obraz i powtórz walidację oraz `docker compose up -d --wait --wait-timeout 300`. Nowe wydanie wymaga zmiany tagu obrazu w `docker-compose.yml`. Przy przywracaniu wcześniejszej wersji użyj także kopii konfiguracji wykonanej przed aktualizacją.
