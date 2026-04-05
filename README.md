# 🌿 PlantGuardian — Smart Grow Box

Chytrý, autonomní box pro dohled a péči o rostlinu. Systém je plně lokální, běží na platformě Home Assistant a využívá mikrokontrolér ESP32 ke čtení ze senzorů a řízení akčních členů. Komunikace probíhá přes ESPHome API a MQTT; webový klient (dashboard) se připojuje přes MQTT WebSockets se zabezpečením TLS/SSL.
###### ANTONÍN REISER, ŠTĚPÁN PRCHAL

---

## ✨ Scénáře automatizace

### 1. 🌡️ Automatické mikroklima
Systém monitoruje teplotu a vlhkost uvnitř boxu pomocí senzoru **DHT22**. Při překročení nastavitelného teplotního prahu se plynule rozbíhá 120mm ventilátor řízený přes **MOSFET (PWM)**.   
Čím vyšší teplota, tím vyšší otáčky - lineární regulace, ne jenom on/off. Threshold lze nastavit fyzicky rotačním enkodérem přímo na krabici nebo z webového dashboardu.

### 2. 🔧 Manuální ovládání klapky / průtoku závlahy
Servo motor ovládá mechanickou klapku (střešní ventilaci nebo ventil). Aktivace probíhá z webového dashboardu nebo fyzickým tlačítkem na krabici.  
Slouží jako manuální zásah do ventilace nebo budoucí příprava pro závlahový systém (s doplněním senzoru vlhkosti půdy).

### 3. 🚨 Bezpečnostní systém
Víceúrovňový alarm hlídající stav boxu:
- **Reed switch modul (KY-025)** - magnet na víku, modul na těle krabice. Otevření víka spustí alarm.
- **Flame sensor** - detekce ohně v blízkosti boxu (relevantní kvůli 12V elektronice a napájecímu zdroji).
- **Rtuťový nakláněcí spínač** - detekce převrácení nebo silného náklonu krabice.
- Při narušení se spustí **pasivní bzučák** (různé tóny pro různé typy alarmu) a na **LCD displeji** bliká varování.
- **Fyzické tlačítko** slouží jako Manual Override - okamžité ztlumení alarmu a přepnutí do servisního režimu.

### 4. 💡 Automatické osvětlení
Senzor intenzity světla (**Light blocking modul**) měří osvětlení uvnitř boxu. Pokud hladina klesne pod nastavený práh (např. zatažená obloha, večer), systém automaticky zapne **Dual White LED pásek** s nezávislou regulací teplé a studené bílé.  
Dva kanály (W1, W2) umožňují přizpůsobit barvu světla fázi růstu rostliny.

---

## 🛠️ Použitý hardware

### 🧠 Centrální jednotka
| Komponenta | Funkce |
|---|---|
| **Raspberry Pi 4B** + MicroSD | Home Assistant OS, Mosquitto MQTT Broker |
| **ESP32 WROOM-32** | Sběr dat ze senzorů, řízení akčních členů (ESPHome) |
| **Sandberg USB Webcam Wide Angle 1080P HD** | Lokální video stream (MotionEye → Generic Camera) |

### 📡 Senzory (vstupy)
| Komponenta | Funkce | Rozhraní |
|---|---|---|
| **DHT22** | Teplota a vlhkost vzduchu | Digitální (GPIO) |
| **KY-025 Reed Switch modul** | Detekce otevření víka boxu | Digitální (D0 → GPIO) |
| **Flame sensor modul** | Detekce ohně / plamene | Digitální (D0 → GPIO) |
| **Rtuťový nakláněcí spínač** | Detekce převrácení boxu | Digitální (GPIO) |
| **[Light blocking modul](https://www.circuitmagic.com/arduino/light-blocking-module-with-arduino-step-by-step-guide/)** | Měření intenzity světla | Digitální |
| **Rotary encoder** | Fyzické nastavení teplotního prahu | Digitální (GPIO — CLK, DT, SW) |
| **Push button** | Manual override / servisní režim | Digitální (GPIO) |

### ⚙️ Akční členy (výstupy)
| Komponenta | Funkce | Řízení |
|---|---|---|
| **120mm Cooler Master ventilátor** | Aktivní odvětrávání boxu | IRF520 + BC547B gate driver, PWM (12V) |
| **Servo motor SM-S2309S** | Mechanická klapka / ventil | PWM signál (5V) |
| **Dual White (CCT) LED pásek (5V)** | Osvětlení pro růst rostliny (teplá + studená bílá) | 2× BC547B, PWM (5V) |
| **Pasivní bzučák** | Akustická signalizace (více tónů) | PWM (3.3V) |
| **LCD displej LCM1602C** | Lokální zobrazení stavu a varování | I²C / paralelně (5V) |

### 🔌 Silová a propojovací elektronika
| Komponenta | Funkce |
|---|---|
| **IRF520 MOSFET** | Spínání 12V větve pro ventilátor (PWM regulace) |
| **3× BC547B NPN tranzistor** | Gate driver pro IRF520 (ventilátor) + přímé spínání LED kanálů W1/W2 |
| **Potenciometr Bochen 3296** | Nastavení kontrastu LCD displeje |
| **Breadboard, propojky, DuPont kabely** | Prototypové zapojení |
| **Externí napájecí adaptér 12V** | Napájení ventilátoru |

---

## 🏗️ Architektura systému

```
┌─────────────────────────────────────────────────────┐
│                  Raspberry Pi 4B                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   │
│  │ Home     │  │ Mosquitto│  │ MotionEye        │   │
│  │ Assistant│  │ MQTT     │  │ (USB kamera)     │   │
│  └────┬─────┘  └────┬─────┘  └──────────────────┘   │
│       │ ESPHome API  │ MQTT                          │
└───────┼──────────────┼───────────────────────────────┘
        │    Wi-Fi     │
┌───────┼──────────────┼───────────────────────────────┐
│       ▼              ▼          ESP32 WROOM-32       │
│  ┌─────────────────────────────────────────────┐     │
│  │              GPIO / ADC / PWM / I²C         │     │
│  └──┬──────┬──────┬──────┬──────┬──────┬───────┘     │
│     │      │      │      │      │      │             │
│  DHT22  KY-025  Flame  Hg-SW  Light  Encoder        │
│         Reed          tilt   block   + Button        │
│                                                      │
│  ┌─────────────────────────────────────────────┐     │
│  │            Výstupy (PWM / GPIO)             │     │
│  └──┬──────────────┬──────┬──────┬─────────────┘     │
│     │              │      │      │                   │
│  BC547B→IRF520  2×BC547B Servo  Buzzer  LCD          │
│  (Fan 12V)    (LED 5V) (Klapka) (Alarm) (Status)    │
└──────┬──────────────┬────────────────────────────────┘
       │              │
  ┌────┴────┐    ┌────┴────┐
  │  12V    │    │   5V    │
  │  Fan    │    │  LED    │
  └─────────┘    └─────────┘
  Společná zem (GND) mezi 3.3V/5V a 12V větvemi
```

**Logická vrstva (3.3V / 5V):** ESP32 čte senzory a řídí logiku. Komunikuje s HA přes Wi-Fi (ESPHome API + MQTT). LED pásek (5V) je spínán přímo přes BC547B tranzistory — nízký odběr (~3 mA/kanál) nevyžaduje MOSFET.

**Výkonová vrstva (12V):** Oddělený napájecí okruh pro ventilátor. BC547B slouží jako gate driver pro IRF520 MOSFET (ESP32 GPIO dává 3.3V, IRF520 potřebuje ~10V na gate). Obě vrstvy sdílejí společnou zem.

---

## 📊 Přehled entit a integrací

### Lokální integrace
| Integrace | Účel |
|---|---|
| **ESPHome** | Komunikace s ESP32, konfigurace senzorů a akčních členů |
| **MQTT (Mosquitto)** | Přenos dat pro webový dashboard (WebSockets) |
| **Generic Camera** | Video stream z USB kamery přes MotionEye |

### Seznam entit
| Entita | Typ | Scénář |
|---|---|---|
| `sensor.temperature` | Senzor | 1 — Mikroklima |
| `sensor.humidity` | Senzor | 1 — Mikroklima |
| `sensor.light_level` | Senzor | 4 — Osvětlení |
| `binary_sensor.box_lid` | Binární senzor | 3 — Bezpečnost |
| `binary_sensor.flame` | Binární senzor | 3 — Bezpečnost |
| `binary_sensor.tilt` | Binární senzor | 3 — Bezpečnost |
| `binary_sensor.override_button` | Binární senzor | 3 — Bezpečnost |
| `sensor.temp_threshold` | Senzor | 1 — Mikroklima |
| `fan.ventilator` | Akční člen | 1 — Mikroklima |
| `light.led_w1` | Akční člen | 4 — Osvětlení |
| `light.led_w2` | Akční člen | 4 — Osvětlení |
| `switch.servo_flap` | Akční člen | 2 — Klapka |
| `switch.buzzer` | Akční člen | 3 — Bezpečnost |
| `camera.plantguardian_cam` | Kamera | Monitoring |

---

## 📋 Implementační plán

### Fáze 1: Infrastruktura ✅
- [x] Instalace Home Assistant OS na Raspberry Pi
- [x] Zprovoznění lokální sítě (RPi ethernet + Wi-Fi pro ESP32)
- [x] Instalace doplňků: Mosquitto broker, ESPHome, File editor
- [x] Integrace USB kamery (MotionEye → Generic Camera)
- [x] Nastavení Git repozitáře

### Fáze 2: Senzory a nízkonapěťové obvody
- [ ] Zapojení a konfigurace ESP32 na breadboardu
- [ ] Připojení DHT22 — ověření čtení teploty a vlhkosti v HA
- [ ] Připojení KY-025 reed switch modulu — detekce otevření víka
- [ ] Připojení flame sensoru, Hg spínače, light blocking modulu
- [ ] Připojení rotary encoderu a push buttonu
- [ ] Konfigurace pasivního bzučáku (PWM tóny)
- [ ] Zápis ESPHome YAML a ověření všech entit v HA

### Fáze 3: Výkonová elektronika a akční členy
- [ ] Zapojení sdíleného GND a 12V napájecího adaptéru
- [ ] IRF520 + BC547B gate driver — PWM řízení ventilátoru
- [ ] 2× BC547B — přímé spínání LED pásku (W1 + W2 kanály)
- [ ] Servo motor — mechanická klapka
- [ ] Konfigurace automatizací v HA (pravidla pro scénáře 1–4)

### Fáze 4: Dashboard, zabezpečení a finalizace
- [ ] Zapojení LCD displeje LCM1602C s potenciometrem
- [ ] Vytvoření vlastního webového klienta (HTML/JS/CSS)
- [ ] Napojení klienta na MQTT přes WebSockets
- [ ] Zabezpečení komunikace (self-signed TLS/SSL certifikát)
- [ ] Autorizace přístupu k dashboardu
- [ ] Montáž do finální krabice

---
