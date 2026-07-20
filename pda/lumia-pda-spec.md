# Lumia 950 XL — Personal Digital Assistant Dashboard

## Specyfikacja projektu v1.1

---

## 1. Przegląd

Projekt zakłada stworzenie lekkiej aplikacji PWA (Progressive Web App) pełniącej rolę osobistego asystenta (PDA) na Microsoft Lumia 950 XL z zainstalowanym Windows on ARM (WoA). Aplikacja obsługuje dwa tryby wyświetlania jednocześnie:

- **Lumia 950 XL (5.7", poziomo)** — panel sterowania z dużymi button boxami
- **Blaupunkt monitor (Full HD, pionowo 1080×1920)** — tablica informacyjna

Lumia stoi na biurku podpięta USB-C do huba (zasilanie + HDMI do monitora). Krótkotrwałe odłączenia (~30 min) są przewidziane.

---

## 2. Przygotowanie urządzenia (prerequisite)

### 2.1 Wymagany system: Windows on ARM (WoA)

Dashboard wymaga nowoczesnej przeglądarki (Edge Chromium ARM64) — oryginalny Windows 10 Mobile NIE obsługuje Service Workers, nowoczesnego JS (fetch, async/await, CSS Grid) ani PWA. Dlatego WoA jest konieczny.

### 2.2 Konfiguracja dual boot

Urządzenie konfigurowane w trybie dual boot: WoA (domyślny) + W10M (zapasowy).

- **WoA** — domyślny boot, obsługuje dashboard, Edge Chromium, HDMI out
- **W10M** — zapasowy boot dla kamery PureView (jedyny system z działającym aparatem)

### 2.3 Procedura instalacji WoA

1. Pobrać WPInternals — odblokować bootloader
2. Pobrać WOA Deployer for Lumia (https://github.com/WOA-Project/WOA-Deployer-Lumia)
3. Wybrać wersję Windows (zalecany: Windows 10 build 19044 / 21h2)
4. Włączyć dual boot (WOA Deployer — 2 kliknięcia)
5. Pobrać i zainstalować sterowniki z Lumia-Drivers (https://github.com/WOA-Project/Lumia-Drivers)
6. Zainstalować Edge Chromium ARM64
7. Zainstalować MobileShell (opcjonalnie — adaptacyjna powłoka)

### 2.4 Weryfikacja hardware po instalacji WoA

Przed uruchomieniem dashboardu sprawdzić:

| Komponent | Status w WoA | Wymagany dla dashboardu |
|---|---|---|
| Wi-Fi | ✅ Działa | Tak — połączenie z API |
| Touchscreen | ✅ Działa | Tak — sterowanie |
| HDMI out | ✅ Działa | Tak — Blaupunkt |
| USB-C (hub) | ✅ Działa | Tak — dock |
| Bluetooth | ✅ Działa | Opcjonalny |
| GPS | ✅ Działa | Opcjonalny (pogoda) |
| NFC | ✅ Działa | Opcjonalny (v2) |
| Audio | ✅ Działa | Opcjonalny (voice) |
| Cellular/LTE | ⚠️ Częściowo | Opcjonalny (backup net) |
| Kamera | ❌ Nie działa | Nie — reboot do W10M |
| Windows Hello | ❌ Nie działa | Nie |

### 2.5 Uwagi dot. baterii

- Oryginalna bateria (3340 mAh, 2015) prawdopodobnie zdegradowana — szukać sprawdzonego zamiennika
- WoA na baterii: ~4h aktywnego użycia (testowane przez community)
- Fast charging działa: 0→50% w 36 min, 0→100% w 1h39min
- Docelowy scenariusz: urządzenie 95% czasu na dock (USB-C zasilanie)
- Unikać nieoryginalnych baterii — powodują niestabilność systemu (ostrzeżenie WoA Project)

---

## 3. Architektura

### 3.1 Stack technologiczny

- **Frontend**: HTML/CSS/JS — pojedynczy plik PWA (lekki, bez frameworka)
- **Backend**: brak dedykowanego — komunikacja bezpośrednio z API
- **AI Asystent**: Claude API (Haiku 4.5 — $1/$5 MTok) z fallbackiem na Sonnet
- **Kalendarz**: Google Calendar API (OAuth2) lub CalDAV
- **Pogoda**: Open-Meteo API (darmowe, bez klucza)
- **Zadania**: localStorage + opcjonalny sync (Notion API lub plik JSON)
- **Hosting**: lokalnie na Lumii (plik .html otwierany w przeglądarce)

### 3.2 Responsywność

Aplikacja wykrywa rozmiar okna i automatycznie przełącza layout:

| Szerokość okna | Tryb | Ekran docelowy |
|---|---|---|
| < 960px landscape | Sterowanie (button box) | Lumia 5.7" |
| < 600px portrait | Kompakt (mini dashboard) | Lumia 5.7" |
| ≥ 600px portrait | Info dashboard | Blaupunkt pion |

Użytkownik może ręcznie przełączyć tryb na Lumii: sterowanie ↔ pełny dashboard.

---

## 4. Ekran Lumia — tryb sterowania (landscape)

### 6.1 Layout

```
┌──────────────────────────────────────────────┐
│ 14:37 · sob. 19 lip          WiFi LTE 73%   │
├──────────────────┬───────────────────────────┤
│ ┌──────┐┌──────┐ │ Następne wydarzenie       │
│ │Asyst.││Kalen.│ │ Sesja zdjęciowa — 14:00   │
│ └──────┘└──────┘ ├───────────────────────────┤
│ ┌──────┐┌──────┐ │ Asystent AI               │
│ │Zadan.││Telef.│ │ "Kolokwium jutro o 9..."  │
│ └──────┘└──────┘ │                           │
├──────────────────┴───────────────────────────┤
│ [Poczta][Aparat][Mapy][Muzyka] [Pełny] ●TV  │
├──────────────────────────────────────────────┤
│ Claude Pro: Sesja 42% │ Tydzień 37% │ F 0%  │
└──────────────────────────────────────────────┘
```

### 4.2 Elementy

- **Button box 2×2**: Asystent, Kalendarz, Zadania, Telefon — duże klikalne kafelki z ikonami (min 80×80px touch target)
- **Info cards**: najbliższe wydarzenie + ostatnia wiadomość asystenta
- **Pasek dolny**: mniejsze ikony (poczta, aparat, mapy, muzyka), przycisk "Pełny dashboard", wskaźnik połączenia z TV
- **Pasek Claude Pro**: 3 paski zużycia — sesja 5h, tydzień, Fable

### 4.3 Tryb pełny (przełączany)

Dwie kolumny: lewa (zegar + kalendarz), prawa (asystent + zadania). Przycisk powrotu do button boxów.

---

## 5. Ekran Blaupunkt — tryb info (portrait)

### 6.1 Layout

```
┌────────────────────┐
│                    │
│      14:37         │
│  sob. 19 lipca     │
│   27°C słonecz.    │
│                    │
├────────────────────┤
│ 📅 Dzisiaj         │
│ ▓ Spotkanie 10-12  │
│ █ Sesja zdj. 14-16 │
│ ░ Kolokwium jutro  │
├────────────────────┤
│ 💬 Asystent        │
│ "Jutro kolokwium   │
│  z zarządzania..." │
├────────────────────┤
│ Claude Pro         │
│ Sesja ████░░ 42%   │
│ Tydz. ███░░░ 37%   │
│ Fable ░░░░░░  0%   │
├────────────────────┤
│ ☑ Do zrobienia     │
│ ○ Zdjęcia Nowakowi │
│ ○ Materiały obóz   │
│ ● Transport ✓      │
└────────────────────┘
```

### 6.2 Zasady

- **Brak interakcji** — ekran czysto informacyjny, aktualizowany automatycznie
- Wydarzenie najbliższe wyróżnione kolorem i obramowaniem
- Wydarzenie zakończone przygaszone (opacity)
- Odświeżanie danych co 60 sekund
- Zegar aktualizowany co sekundę

---

## 6. Moduły funkcjonalne

### 6.1 Asystent AI (Claude)

- **Model domyślny**: Haiku 4.5 (szybki, tani)
- **Eskalacja**: Sonnet 4.6 na dłuższe/złożone pytania (wykrywane po długości inputu lub manualnie)
- **System prompt**: osobisty asystent — zna imię użytkownika, kontekst (student PW, drużynowy FSE, fotograf), styl: krótkie odpowiedzi po polsku
- **Prompt caching**: system prompt cachowany (90% oszczędności)
- **Funkcje**:
  - Odpowiadanie na pytania
  - Podsumowywanie notatek (paste tekstu)
  - Proaktywne przypomnienia (na podstawie kalendarza)
  - Dyktowanie notatek (speech-to-text → tekst → Claude przetwarza)
  - Sugestie zadań
- **Interfejs na Lumii**: pole tekstowe + przyciski szybkich akcji ("Tak", "Przypomnij", "Pytanie")
- **Interfejs na Blaupunkcie**: ostatnia odpowiedź asystenta (read-only)
- **Klucz API**: wpisywany w ustawieniach, przechowywany w localStorage

### 6.2 Kalendarz

- **Źródło**: Google Calendar API (OAuth2) lub iCal URL (publiczny link .ics)
- **Prostszy wariant na start**: publiczny link .ics parsowany lokalnie (bez OAuth)
- **Wyświetlanie**: dzisiejsze i jutrzejsze wydarzenia
- **Kolorowanie**: po statusie (zakończone, nadchodzące, przyszłe)
- **Odświeżanie**: co 5 minut

### 6.3 Pogoda

- **API**: Open-Meteo (darmowe, bez rejestracji)
- **Dane**: temperatura aktualna, opis, ikona
- **Lokalizacja**: hardcoded (Ożarów Mazowiecki) lub GPS
- **Odświeżanie**: co 30 minut

### 6.4 Zadania (todo)

- **Przechowywanie**: localStorage (offline-first)
- **Opcjonalny sync**: Notion API lub prosty plik JSON na Google Drive
- **Operacje**: dodaj, oznacz jako zrobione, usuń
- **Interakcja**: tylko z Lumii (checkbox tap)
- **Wyświetlanie na Blaupunkcie**: lista read-only

### 6.5 Zużycie Claude Pro

- **Źródło danych**: brak publicznego API — dane ręcznie wpisywane lub scrapowane z claude.ai (kruche)
- **Alternatywa**: estymacja na podstawie liczby wysłanych wiadomości w sesji dashboardu
- **Wyświetlanie**: 3 paski postępu (sesja, tydzień, Fable)

### 6.6 Zegar i data

- **Zegar**: aktualizowany co sekundę, duża czcionka na Blaupunkcie
- **Data**: dzień tygodnia, data, formatowanie polskie
- **Strefa czasowa**: Europe/Warsaw

---

## 7. Funkcje dodatkowe (v2)

Poniższe mogą być dodane iteracyjnie:

- **Voice input**: Web Speech API (speech-to-text) → wiadomość do Claude
- **NFC tagi**: trigger akcji (wymaga natywnej apki — poza zakresem PWA)
- **USB OTG**: przeglądanie plików z karty SD (wymaga natywnego file pickera)
- **Pomodoro timer**: widoczny na monitorze, sterowany z telefonu
- **RSS feed**: przewijane newsy na Blaupunkcie
- **Home Assistant**: sterowanie IoT przez API
- **Obsidian sync**: wyświetlanie notatek z vaulta
- **Powiadomienia SMS**: wyświetlanie na Blaupunkcie (wymaga integracji z cellular stackiem WoA)

---

## 8. Wymagania techniczne

### 8.1 Środowisko docelowe

| Parametr | Wartość |
|---|---|
| Urządzenie | Microsoft Lumia 950 XL (Cityman) |
| SoC | Qualcomm Snapdragon 810 (MSM8994) |
| RAM | 3 GB |
| System | Windows 10/11 ARM64 (WoA) |
| Przeglądarka | Edge Chromium ARM64 lub Firefox ARM64 |
| Ekran telefonu | 5.7" 1440×2560 AMOLED |
| Monitor zewnętrzny | Blaupunkt Full HD w pionie (1080×1920) |
| Połączenie | USB-C hub (HDMI + zasilanie) |
| Sieć | Wi-Fi + LTE (SIM) |

### 8.2 Ograniczenia

- Brak kamery na custom OS
- Bateria ~4h aktywnego użycia (urządzenie głównie na dock)
- Snap 810 throttluje pod obciążeniem — JS musi być lekki
- Brak gwarancji stabilności cellular na nowszych buildach WoA
- localStorage jedyny pewny storage (brak gwarancji IndexedDB na starszym Edge)

### 8.3 Wydajność

- Pierwszy render < 2 sekundy
- Bundle < 200 KB (bez frameworka)
- Brak ciężkich animacji (oszczędność baterii i CPU)
- AMOLED: preferowany ciemny motyw (piksele wyłączone = oszczędność energii)
- Minimalne użycie DOM — unikanie re-renderów

---

## 9. Struktura plików

```
lumia-pda/
├── index.html          # główny plik PWA (all-in-one)
├── manifest.json       # PWA manifest
├── sw.js               # service worker (offline cache)
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
└── README.md
```

Docelowo **jeden plik HTML** z osadzonym CSS i JS — minimalizacja requestów, działa offline po pierwszym załadowaniu.

---

## 10. Konfiguracja użytkownika (Settings)

Dostępne z przycisku "Ustawienia" na Lumii:

- **Klucz Claude API**: input text (maskowany)
- **Model domyślny**: Haiku / Sonnet (dropdown)
- **URL kalendarza iCal**: input text
- **Lokalizacja pogody**: input text lub "użyj GPS"
- **Motyw**: ciemny (domyślny) / jasny
- **Imię użytkownika**: do personalizacji system promptu asystenta

---

## 11. Estymacja kosztów miesięcznych

| Składnik | Koszt |
|---|---|
| Claude Haiku (50 msg/dzień, ~500 tok) | ~$2–4 |
| Open-Meteo | $0 (darmowe) |
| Google Calendar (iCal) | $0 |
| Hosting | $0 (lokalne) |
| **Razem** | **~$2–4/miesiąc** |

---

## 12. MVP — zakres pierwszej wersji

1. ✅ Pojedynczy plik HTML (PWA-ready)
2. ✅ Responsywny layout (Lumia landscape / Blaupunkt portrait)
3. ✅ Button box sterowanie na Lumii
4. ✅ Przełączanie tryb sterowania ↔ pełny dashboard
5. ✅ Zegar i data (pl)
6. ✅ Pogoda z Open-Meteo
7. ✅ Czat z Claude (Haiku API)
8. ✅ Lista zadań (localStorage)
9. ✅ Ciemny motyw domyślny (AMOLED)
10. ✅ Offline fallback (service worker)

### Poza MVP (v2+)

- Kalendarz Google (iCal parse)
- Voice input
- Pomodoro timer
- Estymacja zużycia Claude Pro
- RSS feed
- Home Assistant

---

*Specyfikacja przygotowana: 19 lipca 2026*
*Urządzenie docelowe: Microsoft Lumia 950 XL + Blaupunkt FHD portrait*
