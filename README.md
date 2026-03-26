# Sarkastický Spolubydlící (a.k.a. Google Crib)

> **Context-Aware AI Assistant**  
> *„Protože obyčejný termostat se tě nezeptá, jestli se snažíš simulovat dobu ledovou, nebo jsi jen líný zapnout topení.“*

## 📋 Přehled projektu

Tento projekt transformuje **Raspberry Pi 4B** na pokročilého domácího asistenta s osobností. Na rozdíl od komerčních řešení tento systém **vidí**, **slyší** a **cítí** (teplotu). Využívá distribuovanou architekturu (RPi jako server + ESP32 jako senzor) a LLM pro generování kousavých poznámek k stavu domácnosti.

### Klíčové funkce

* **🔊 Voice Activation (Local):** Naslouchá na klíčové slovo (Wake Word) lokálně.
* **🧠 Visual Critique:** Webkamera autonomně sleduje nepořádek a komentuje ho.
* **🌡️ Thermal & Environmental Bullying:** Satelitní jednotka ESP32 monitoruje mikroklima. Pokud teplota vybočí z komfortní zóny, asistent začne být nepříjemný, dokud uživatel nevyvětrá nebo nezatopí.
* **💬 Latency Masking:** Využití "výplňových frází" pro maskování odezvy cloudu.

---

## 🛠 Hardware Stack

### Centrální Jednotka (The Brain)

* **SBC:** Raspberry Pi 4B (4GB RAM) – *[Vlastní zdroje]*
* **Vstup (Audio):** USB Všesměrový mikrofon
* **Výstup (Audio):** Aktivní reproduktor (3.5mm Jack / USB)
* **Vstup (Video):** USB Webkamera (Logitech C270 nebo ekvivalent)

### Satelitní Jednotka (The Skin)

* **MCU:** ESP32 DevKit V1 (komunikace přes Wi-Fi / ESPHome)
* **Senzor:** DHT22 nebo BME280 (Teplota + Vlhkost)
* **Napájení:** 5V USB adaptér (nezávislé na RPi)

---

## 🧩 Software Architecture

Systém je postaven na platformě **Home Assistant (Supervised)**.

### 1. Audio Pipeline (Ears & Mouth)

* **Wake Word:** `OpenWakeWord` (lokálně na RPi).
* **STT/TTS:** Hybridní přístup (Cloud pro kvalitu, Local cache pro rychlost).

### 2. Environmental Awareness (IoT Layer)

Teplota není měřena přímo na RPi (kvůli ovlivnění teplem procesoru), ale na externím **ESP32** modulu pomocí protokolu **ESPHome API**.

* ESP32 zasílá data každých 60s nebo při změně o 0.5 °C.
* Home Assistant tato data vyhodnocuje a spouští "Komfortní AI rutinu".

### 3. The Brain (LLM Logic)

`Gemini 3 Flash` dostává kontextová data složená z:

* **Vizuálu:** Snapshot z kamery.
* **Senzoriky:** Aktuální teplota a vlhkost z ESP32.
* **Prompt:** *"Je tu 17 stupňů a na fotce sedí člověk v mikině. Vynadej mu, že šetří na topení, a buď sarkastický."*

---

## 🚀 Instalace a Konfigurace

### Krok 1: Příprava HW

1. Instalace **Home Assistant OS** na RPi (vlastní HW).
2. Flashnutí **ESPHome** firmwaru do ESP32 s konfigurací pro senzor teploty.

### Krok 2: Automatizace (YAML ukázka)

**Logika "Thermal Critique":**

```yaml
alias: "Temperature Rant"
trigger:
  - platform: numeric_state
    entity_id: sensor.esp32_room_temperature
    above: 26
    for: "00:10:00"
  - platform: numeric_state
    entity_id: sensor.esp32_room_temperature
    below: 18
    for: "00:10:00"
action:
  - service: openai_conversation.process
    data:
      text: >
        Teplota v pokoji je {{ states('sensor.esp32_room_temperature') }} stupňů.
        Je to extrém. Vymysli urážlivou poznámku o tom, že buď žijeme v sauně, nebo v lednici.
        Odkazuj na moji neschopnost ovládat termostat.
      agent_id: conversation.sarcastic_bot
    response_variable: ai_response
  - service: tts.google_say
    data:
      message: "{{ ai_response.response.text }}"

```

---

## 🖥 Web Dashboard

Jednoduché rozhraní běžící na `http://rpi-ip:8123/dashboard-chat`.

* Graf vývoje teploty (historie z ESP32).
* Chatovací okno s historií urážek.
* Tlačítko "Emergency Silence" (vypne reproduktor).

---

## ⚠️ Technické poznámky

* **Proč ESP32?** Oddělení senzoru od RPi zajišťuje přesnější měření (eliminace tepla z CPU RPi) a demonstruje síťovou komunikaci v rámci IoT zadání.
* **Vlastní zdroje:** RPi 4B použité v projektu je soukromým majetkem studenta a slouží jako aplikační server. Školní/zakoupené komponenty jsou ESP32 a senzory.

---

*Autoři: Antonín Reiser a Štěpán Prchal
