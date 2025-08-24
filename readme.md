# Kompleksowa Automatyzacja Infrastruktury Domowej z Ansible

Ten projekt demonstruje zaawansowane wykorzystanie Ansible do w pełni zautomatyzowanego zarządzania, konfiguracji i utrzymania zróżnicowanej infrastruktury domowej (Homelab). Rozwiązanie, oparte na filozofii **Infrastructure as Code (IaC)**, przekształca ręczne i podatne na błędy zadania administracyjne w idempotentne, powtarzalne i bezpieczne playbooki.

## Kluczowe Osiągnięcia i Funkcjonalności

*   **Zarządzanie Heterogenicznym Środowiskiem:** Automatyzacja obejmuje standardowe serwery Linux (Debian-based), specjalistyczne urządzenia typu "appliance" (TrueNAS SCALE) oraz sprzęt sieciowy (Mikrotik RouterOS).
*   **Architektura Oparta na Rolach:** Cała logika została zamknięta w modułowych i reużywalnych rolach, co zapewnia czystość kodu i promuje zasadę **Don't Repeat Yourself (DRY)**.
*   **Bezpieczne Zarządzanie Sekretami:** Wszystkie wrażliwe dane (hasła, klucze API) są centralnie zarządzane i szyfrowane w jednym pliku **Ansible Vault**, eliminując ryzyko przechowywania sekretów w postaci jawnego tekstu.
*   **Idempotentne Operacje:** Playbooki są zaprojektowane tak, aby można je było wielokrotnie uruchamiać bez niepożądanych skutków ubocznych. Wdrożono oddzielne role do **sprawdzania** stanu systemów i **aplikowania** aktualizacji.
*   **Automatyzacja Sterowana API:** Aktualizacje dla TrueNAS SCALE są realizowane poprzez interakcję z jego **REST API**, co pokazuje zdolność do integracji z nowoczesnymi interfejsami programistycznymi.
*   **Mechanizmy Bezpieczeństwa:** Kluczowe operacje, takie jak restart urządzeń sieciowych, są zabezpieczone przez interaktywną pauzę, wymagającą ręcznego potwierdzenia przez operatora.
*   **Orkiestracja Wielu Systemów:** Główny playbook (`run_all_updates.yml`) pełni rolę centralnego punktu sterowania (single point of control), który sekwencyjnie zarządza aktualizacjami w całej infrastrukturze.

## Struktura Projektu

Projekt jest zorganizowany zgodnie z najlepszymi praktykami Ansible, co zapewnia jego skalowalność i łatwość w utrzymaniu.

```
.
├── ansible.cfg                 # Główny plik konfiguracyjny, definiujący ścieżki
├── group_vars/
│   ├── all/
│   │   └── vault.yml           # CENTRALNY, ZASZYFROWANY PLIK Z SEKRETAMI
├── hosts.ini                   # Plik inwentarza, grupujący hosty wg funkcji i typu OS
├── playbooks/
│   ├── run_updates.yml     # Główny playbook orkiestrujący wszystkie aktualizacje
│   └── ...                     # Inne playbooki operacyjne
├── roles/
│   ├── update_debian_systems/  # Rola do APLIKOWANIA aktualizacji na Debianie (moduł apt)
│   ├── update_truenas/         # Rola do aktualizacji TrueNAS przez REST API (moduł uri)
│   └── update_mikrotik/        # Rola do aktualizacji Mikrotik RouterOS (kolekcja community.routeros)
└── README.md                   # Ten plik
```

## Główne Komponenty

### Inwentarz (`hosts.ini`)
Infrastruktura jest logicznie podzielona na grupy, takie jak `[debian_servers]`, `[truenas_servers]` i `[mikrotik]`. Taka struktura pozwala na precyzyjne targetowanie operacji tylko do odpowiednich maszyn.

### Zarządzanie Sekretami (Ansible Vault)
Wszystkie sekrety (klucz API TrueNAS, hasło do Mikrotika) są przechowywane w jednym, zaszyfrowanym pliku `group_vars/all/vault.yml`. Role odwołują się do tych zmiennych w sposób ustrukturyzowany (np. `{{ vault_secrets.truenas.api_key }}`), co zapewnia bezpieczeństwo i centralizację zarządzania.

### Role
*   **`update_debian_systems`**: Wykorzystuje moduł `ansible.builtin.apt` do pełnej aktualizacji systemu, usuwania niepotrzebnych zależności oraz sprawdzania, czy wymagany jest restart.
*   **`update_truenas`**: Komunikuje się z REST API TrueNAS SCALE za pomocą modułu `ansible.builtin.uri`. Realizuje proces sprawdzania dostępności aktualizacji, ich pobierania i instalacji, a na końcu bezpiecznego restartu urządzenia.
*   **`update_mikrotik`**: Używa modułów z kolekcji `community.routeros` do wykonywania poleceń bezpośrednio w terminalu RouterOS. Posiada wbudowany mechanizm pauzy przed restartem krytycznego elementu sieci.

## Użycie

### Wymagania Wstępne
1.  Zainstalowany Ansible na maszynie kontrolnej.
2.  Zainstalowana kolekcja Ansible dla RouterOS:
    ```bash
    ansible-galaxy collection install community.routeros
    ```

### Konfiguracja
1.  **Stwórz plik Vault:** Skopiuj strukturę z `group_vars/all/vault.yml` i uzupełnij swoimi sekretami, używając polecenia:
    ```bash
    ansible-vault create group_vars/all/vault.yml
    ```
2.  **Dostosuj inwentarz:** Zaktualizuj plik `hosts.ini`, wpisując adresy IP i nazwy hostów swojej infrastruktury.

### Uruchomienie
Aby uruchomić pełny proces aktualizacji całej infrastruktury, wykonaj polecenie:```bash
ansible-playbook playbooks/run_all_updates.yml --ask-vault-pass -K
```
*   `--ask-vault-pass`: Poprosi o hasło do głównego pliku Vault.
*   `-K` (`--ask-become-pass`): Poprosi o hasło `sudo` dla operacji wymagających podniesionych uprawnień.

## Zaprezentowane Koncepcje i Umiejętności

Ten projekt jest praktyczną demonstracją następujących kluczowych kompetencji z obszaru DevOps i Administracji Systemami:

*   **Infrastructure as Code (IaC):** Pełna definicja i zarządzanie infrastrukturą za pomocą kodu.
*   **Configuration Management:** Utrzymanie spójnego i zdefiniowanego stanu na wszystkich zarządzanych węzłach.
*   **Idempotency:** Gwarancja, że wielokrotne uruchomienie
