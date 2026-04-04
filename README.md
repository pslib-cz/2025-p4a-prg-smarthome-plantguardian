# 🌿 PlantGuardian IoT - Smart Grow Box

**PlantGuardian** je školní IoT projekt (případová studie) pro dvoučlenný tým. Jedná se o chytrý, autonomní box pro dohled a péči o rostlinu/skleník. Systém je plně lokální, běží na platformě Home Assistant a využívá mikrokontrolér ESP32 ke čtení ze senzorů a řízení akčních členů.

Cílem projektu je demonstrovat reálné nasazení IoT technologií s důrazem na minimalizaci cloudových závislostí a maximální lokální bezpečnost.

---

## ✨ Funkce a Scénáře (Případová studie)

Projekt obsahuje tři hlavní automatizační scénáře, které simulují reálné chování chytrého spotřebiče:

1. **🌡️ Automatické mikroklima (Dynamické chlazení):** 
   - Systém monitoruje teplotu a vlhkost pomocí senzoru DHT22.
   - Při překročení stanovené meze je pomocí PWM signálu a MOSFET tranzistoru plynule spuštěn 120mm ventilátor pro odsávání vzduchu.
2. **💧 Inteligentní závlaha / Fyzická klapka:**
   - Na základě pokynu z webového dashboardu nebo fyzického tlačítka se otočí servo motor, který uvolní ventil/otevře střešní klapku.
3. **🚨 Bezpečnostní Alarm a Override:**
   - Senzor otevření víka (mikrospínač) hlídá neoprávněný přístup. Pokud dojde k narušení, spustí se akustický alarm (bzučák) a na LCD displeji začne blikat varování.
   - Lokální fyzické tlačítko slouží jako *Manual Override* pro okamžité ztlumení poplachu nebo ruční spuštění ventilace.

---

## 🛠️ Použitý Hardware

Projekt je sestaven převážně z recyklovaných a běžně dostupných elektronických součástek.

### 🧠 Centrální jednotka a Komunikace
* **Raspberry Pi 4B** + MicroSD karta (Hostuje Home Assistant OS a MQTT Broker)
* **ESP32 WROOM-32** (Sběr dat a řízení HW přes ESPHome)
* **USB Webkamera Logitech C270** (Lokální videostream)

### 📡 Senzory (Vstupy)
* **DHT22** - Senzor teploty a vlhkosti vzduchu
* **Omron mikrospínač** - Detekce otevření boxu
* **Spínací tlačítko (Push button)** - Fyzický zásah do systému (Manual override)

### ⚙️ Akční členy (Výstupy)
* **120mm Větrák Cooler Master** (Aktivní odvětrávání)
* **Servo motor SM-S2309S** (Mechanická klapka)
* **RGB LED pásek (3M)** (Osvětlení pro růst rostliny)
* **Piezo Bzučák** (Akustická signalizace)
* **LCD Displej LCM1602C** (Lokální zobrazení stavu)

### 🔌 Silová a propojovací elektronika
* **MOSFET IRF520** (Spínání 12V větve pro ventilátor)
* **Relé modul SRD-05VDC-SL-C** (Spínání 12V větve pro LED pásek)
* **Potenciometr Bochen 3296** (Řízení kontrastu LCD displeje)
* **Breadboard a propojky** (Stavebnice Boffin)
* **Externí napájecí adaptér 12V** (Pro napájení ventilátoru a osvětlení)

---

## 🏗️ Architektura a Zapojení systému

Systém využívá dvouúrovňovou architekturu:
1. **Logická vrstva (5V/3.3V):** RPi a ESP32 řídí logiku. Senzory a LCD běží na této nízkonapěťové větvi. Data putují přes lokální Wi-Fi sít do Home Assistanta (protokol ESPHome API a MQTT).
2. **Výkonová vrstva (12V):** Protože RPi a ESP32 nedokážou napájet výkonné členy, je zde oddělený 12V okruh ovládaný skrze MOSFET (PWM regulace) a Relé (binární on/off). Obě vrstvy sdílejí **společnou zem (GND)**.

---

## 📋 Implementační Plán

Vývoj projektu je rozdělen do 4 logických fází:

### Fáze 1: Založení infrastruktury
- [ ] Instalace Home Assistant OS na Raspberry Pi.
- [ ] Zprovoznění lokální Wi-Fi komunikace.
- [ ] Instalace doplňků: Mosquitto broker (MQTT), ESPHome, File editor.
- [ ] Integrace USB kamery přes *Generic Camera* integraci.

### Fáze 2: Sběr dat a nízkonapěťové obvody
- [ ] Zapojení a konfigurace ESP32 na breadboardu.
- [ ] Připojení DHT22, Omron spínače a tlačítka JB k ESP32.
- [ ] Zápis `yaml` kódu do ESPHome a ověření čtení hodnot v HA.
- [ ] Zapojení a konfigurace Piezo bzučáku pro alarm.

### Fáze 3: Výkonová elektronika a akční členy
- [ ] Zapojení sdíleného GND a 12V adaptéru.
- [ ] Integrace MOSFETu IRF520 pro řízení Cooler Master ventilátoru (nastavení PWM v ESPHome).
- [ ] Integrace Relé pro ovládání RGB LED pásku.
- [ ] Konfigurace servomotoru.

### Fáze 4: Dashboard, Zabezpečení a Prezentace
- [ ] Zapojení LCD displeje LCM1602C s potenciometrem Bochen pro lokální výpis logů.
- [ ] Vytvoření vlastního webového klienta (HTML/JS/CSS).
- [ ] Napojení klienta na MQTT přes WebSockets.
- [ ] Zabezpečení komunikace a implementace TLS/SSL autorizace.
- [ ] Zabalení a montáž do finální fyzické krabice.

---

## 🎯 Plnění školního zadání (Hodnocení)

Tento projekt byl navržen tak, aby přesně splňoval požadavky pro **N = 2** (dvoučlenný tým):

* **Lokální integrace (Min. 1):** Projekt využívá minimálně 3 (ESPHome, MQTT, Generic Camera).
* **Počet scénářů (Min. 2):** Projekt obsahuje 3 ucelené scénáře (Chlazení, Závlaha, Bezpečnost).
* **Počet entit (Min. 4):** Systém exportuje 8+ entit (Teplota, Vlhkost, Otáčky větráku, Úhel serva, Stav víka, Tlačítko, Bzučák, Relé světla).
* **Dashboard a monitoring:** Webový klient vytvořen na míru, připojen lokálně s TLS zabezpečením.
* **Absence Cloud API:** Veškerá logika a komunikace je přísně lokální (No-Cloud approach).****
