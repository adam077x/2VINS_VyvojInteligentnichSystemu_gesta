# 2VINS_VyvojInteligentnichSystemu_gesta

![Gesta](/img/gesta.png)

## 1. Požadavky

Pro běh programu je nutné mít nainstalovaný **Python 3.10 nebo 3.11** (doporučeno).

### Použité knihovny
*   **opencv-python**: Práce s obrazem z webkamery.
*   **mediapipe (verze 0.10.9)**: Detekce ruky a kloubů (verze 0.10.9 je vyžadována pro správnou funkčnost modulu `solutions`).
*   **pycaw**: Ovládání audio rozhraní Windows (Core Audio Windows Library).
*   **comtypes**: Závislost pro komunikaci s Windows API.
*   **numpy**: Matematické operace (lineární interpolace).
*   **protobuf (<5)**: Kompatibilní verze protocol buffers pro MediaPipe.

## 2. Instalace

Postup pro čistou instalaci na systému Windows:

1.  **Vytvoření virtuálního prostředí:**
    Otevřete příkazový řádek (CMD nebo PowerShell) ve složce projektu a spusťte:
    ```bash
    python -m venv .venv
    ```

2.  **Aktivace prostředí:**
    *   PowerShell: `.\.venv\Scripts\Activate.ps1`
    *   CMD: `.\.venv\Scripts\activate.bat`

3.  **Instalace závislostí:**
    Nainstalujte knihovny s důrazem na specifické verze pro stabilitu:
    ```bash
    pip install -r requirements
    ```

## 3. Popis kódu (`main.py`)

Kód funguje v nekonečné smyčce, která čte snímky z webkamery a analyzuje je.

### Inicializace
*   Importujeme potřebné knihovny (`cv2`, `mediapipe`, `pycaw` atd.).
*   Konfigurujeme **MediaPipe Hands** s parametry:
    *   `max_num_hands=1`: Detekujeme pouze jednu ruku pro jednodušší ovládání.
    *   `min_detection_confidence=0.7`: Vyžadujeme 70% jistotu detekce.
*   Inicializujeme **Pycaw** pro získání přístupu k `EndpointVolume` (rozhraní pro hlasitost).

### Hlavní smyčka
1.  **Načtení obrazu:** `cap.read()` získá aktuální snímek.
2.  **Detekce:** Obraz se převede na RGB a předá se MediaPipe (`hands.process()`).
3.  **Zpracování landmarks (kloubů):**
    *   Pokud je detekována ruka, projdeme seznam 21 bodů (landmarks).
    *   Zajímají nás body:
        *   **ID 4 (Palec - Thumb Tip)**
        *   **ID 8 (Ukazováček - Index Finger Tip)**
4.  **Výpočet:**
    *   Získáme souřadnice (x, y) těchto bodů.
    *   Vypočítáme Euklidovskou vzdálenost mezi nimi (`math.hypot`).
    *   Střed úsečky mezi prsty slouží pro vykreslení vizuální indikace.
5.  **Ovládání hlasitosti:**
    *   Vzdálenost mezi prsty (typicky 50-250 pixelů) se přepočítá na rozsah hlasitosti (typicky -65 dB až 0 dB) pomocí `numpy.interp`.
    *   Nastavíme hlasitost: `volume.SetMasterVolumeLevel(vol, None)`.
6.  **Vykreslení:**
    *   Vykreslíme body na ruce, čáru mezi prsty a sloupec indikující hlasitost.
    *   Vypočítáme a zobrazíme FPS (počet snímků za sekundu).

## 4. Ovládání

*   **Pinch (Špetka):** Přiblížením palce a ukazováčku snižujete hlasitost.
*   **Rozevření prstů:** Oddálením prstů zvyšujete hlasitost.
*   **Ukončení:** Stisknutím klávesy `q`.

## 5. Známé problémy a jejich řešení

Během vývoje jsme narazili na několik technických komplikací, které jsou v této finální verzi vyřešeny:

### A. Chyba `AttributeError: module 'mediapipe' has no attribute 'solutions'`
*   **Příčina:** Nejnovější verze knihovny MediaPipe (0.10.31) měla v době vývoje chybu (nebo změnu API), která znepřístupnila modul `solutions`.
*   **Řešení:** Program vyžaduje starší, stabilní verzi `mediapipe==0.10.9`.

### B. Nekompatibilita `protobuf`
*   **Příčina:** MediaPipe vyžaduje starší verzi knihovny Protobuf. Novější verze (5.x) způsobují chyby při načítání.
*   **Řešení:** Je nutné instalovat `protobuf<5` (např. verze 4.x).

### C. Změna rozhraní `pycaw` (Chyba `AttributeError: Activate`)
*   **Příčina:** Nainstalovaná verze `pycaw` vracela z metody `GetSpeakers()` rovnou objekt s vlastností `EndpointVolume`, zatímco starší dokumentace odkazovala na metodu `Activate()`.
*   **Řešení:** Kód využívá modernější přístup `volume = devices.EndpointVolume`.

### D. Poškozené virtuální prostředí
*   **Příčina:** Původní prostředí `.venv` se během instalací poškodilo (chybějící cesty v `sys.path`).
*   **Řešení:** Doporučuje se vždy instalovat do čistého prostředí (v našem případě jsme použili `.env`).



