# 🌿 PlantGuardian - Smart Grow Box

Jedná se o chytrý, autonomní box pro dohled a péči o rostlinu. Systém je plně lokální, běží na platformě Home Assistant a využívá mikrokontrolér ESP32 ke čtení ze senzorů a řízení akčních členů.

---

## ✨ Funkce a scénáře (Případová studie)

1. **🌡️ Automatické mikroklima (Dynamické chlazení):** 
   - Systém monitoruje teplotu a vlhkost pomocí DHT22.
   - Při překročení stanovené meze je pomocí MOSFET tranzistoru plynule spuštěn 120mm ventilátor pro odsávání vzduchu. (lepší než relé protože pwm) 
2. **💧 Inteligentní závlaha / Fyzická klapka:**
   - Na základě pokynu z webového dashboardu nebo fyzického tlačítka se otočí servo, který uvolní ventil/otevře střešní klapku/něco prostě bude dělat.
3. **🚨 Bezpečnostní Alarm a Override:**
   - Senzor otevření víka (reed sw) hlídá neoprávněný přístup. Pokud dojde k narušení, spustí se akustický alarm (bzučák) a na LCD displeji začne blikat varování.
   - tlačitko okamžitě ztlumí poplach a zapne maintenance mód (kterej nevim co ještě bude pořádně dělat ale je to integrace).

---

## 🛠️ Použitý HW

### 🧠 Centrální jednotka a Komunikace
* **Raspberry Pi 4B** + MicroSD karta (Hostuje Home Assistant OS a MQTT Broker)
* **ESP32 WROOM-32** (Sběr dat a řízení HW přes ESPHome)
* **USB Webkamera Sandberg Webcam Wide Angle 1080P HD** (Lokální videostream)

### 📡 Senzory (Vstupy)
* **DHT22** - Senzor teploty a vlhkosti vzduchu
* **KY-025 Reed switch** - Maintenance mód - otevření boxu
* **Spínací tlačítko (Push button)** - Fyzický zásah do systému (Manual override) !!!! TOHLE JE NĚJAKÝ DIVNÝ

### ⚙️ Akční členy (Výstupy)
* **120mm Cooler Master větríl** (Aktivní odvětrávání)
* **Servo motor SM-S2309S** (Mechanická klapka)
* **RGB LED pásek (3M)** (Osvětlení pro růst rostliny)
* **Piezo Bzučák** (Akustická signalizace)
* **LCD Displej LCM1602C** (Lokální zobrazení stavu)

### 🔌 Silová a propojovací elektronika
* **MOSFET IRF520** (Spínání 12V větve pro ventilátor)
* **Relé modul SRD-05VDC-SL-C** (Spínání 12V větve pro LED pásek - ale ten podle mě nepotřebuje 12V. nějak zjistit)
* **Potenciometr Bochen 3296** (Řízení kontrastu LCD displeje)
* **Breadboard a propojky a kabely a duponty a hodně věcí**
* **Externí napájecí adaptér 12V** (Pro napájení ventilátoru a osvětlení)

---

## 🏗️ Architektura a zapojení

Systém využívá dvouúrovňovou architekturu:
1. **Logická vrstva (5V/3.3V):** RPi a ESP32 řídí logiku. Senzory a LCD běží na této nízkonapěťové větvi. Data putují přes lokální Wi-Fi sít do Home Assistanta (protokol ESPHome API a MQTT).
2. **Výkonová vrstva (12V):** Protože RPi a ESP32 nedokážou napájet výkonné členy, je zde oddělený 12V okruh ovládaný skrze MOSFET (PWM regulace) a Relé (binární on/off). Obě vrstvy sdílejí **společnou zem (GND)**.

---

## 📋 Implementační plán

Vývoj projektu je rozdělen do 4 logických fází:

### Fáze 1: Založení infrastruktury
- [x] Instalace Home Assistant OS na Raspberry Pi.
- [ ] Zprovoznění lokální Wi-Fi komunikace.
- [x] Instalace doplňků: Mosquitto broker (MQTT), ESPHome, File editor.
- [x] Integrace USB kamery přes *Generic Camera* integraci.

### Fáze 2: Sběr dat a nízkonapěťové obvody
- [ ] Zapojení a konfigurace ESP32 na breadboardu.
- [ ] Připojení DHT22, JB spínače a tlačítka JB k ESP32.
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
