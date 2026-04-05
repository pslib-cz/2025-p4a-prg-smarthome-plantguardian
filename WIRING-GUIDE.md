# ⚡ PlantGuardian — Elektrické zapojení

Tento dokument popisuje kompletní elektrické zapojení všech komponent projektu PlantGuardian.

---

## 🔌 Napájení — Přehled

Systém používá **dva oddělené napájecí zdroje** se společnou zemí (GND):

```
┌────────────────────┐         ┌──────────────────────────────────────┐
│  USB nabíječka /   │         │  12V/1A adaptér                      │
│  USB z RPi         │         │                                      │
└────────┬───────────┘         └────────────┬─────────────────────────┘
         │ USB kabel                        │ DC konektor
         ▼                                 ▼
┌────────────────────┐         ┌──────────────────────────────────────┐
│  ESP32 USB port    │         │  Female DC jack (standalone)         │
│                    │         │  (3 nožičky — použít + a GND)        │
│  Výstupy:          │         │                                      │
│  • 3.3V pin        │         │  Výstup:                             │
│  • 5V / VIN pin    │         │  • +12V drát → breadboard 12V rail   │
└────────┬───────────┘         │  • GND drát → breadboard GND rail    │
         │                     └────────────┬─────────────────────────┘
         │                                  │
         ▼                                  ▼
┌──────────────────────────────────────────────────────────────────────┐
│                        BREADBOARD                                    │
│                                                                      │
│  Rail A: 3.3V (z ESP32 3.3V pinu)     Rail C: +12V (z DC jacku)     │
│  Rail B: 5V (z ESP32 VIN pinu)        Rail D: GND (společná zem)    │
│                                                                      │
│  ⚠️  GND z ESP32 a GND z 12V adaptéru MUSÍ být propojené!          │
└──────────────────────────────────────────────────────────────────────┘
```

**Kolébkový spínač** — zařaď do +12V větve jako hlavní vypínač silového okruhu (mezi DC jack a breadboard).

---

## 🧩 Rozložení GPIO pinů ESP32 WROOM-32

### Vstupy (senzory)

| GPIO | Komponenta | Typ signálu | Poznámka |
|------|-----------|-------------|----------|
| GPIO4 | DHT22 (DATA) | Digitální | Vyžaduje 4.7kΩ pull-up na 3.3V |
| GPIO13 | KY-025 Reed switch (D0) | Digitální | Detekce otevření víka |
| GPIO14 | Flame sensor (D0) | Digitální | Detekce plamene |
| GPIO27 | Hg tilt switch (S) | Digitální | Detekce náklonu |
| GPIO34 | Light blocking (S) | Digitální | ⚠️ Input-only pin, bez pull-up |
| GPIO35 | Push button (S) | Digitální | ⚠️ Input-only pin, bez pull-up |
| GPIO16 | Rotary encoder (CLK) | Digitální | Otáčení — takt |
| GPIO17 | Rotary encoder (DT) | Digitální | Otáčení — směr |
| GPIO5 | Rotary encoder (SW) | Digitální | Stisk encoderu |

### Výstupy (akční členy)

| GPIO | Komponenta | Typ signálu | Poznámka |
|------|-----------|-------------|----------|
| GPIO25 | Ventilátor (→ BC547B → IRF520) | PWM | Gate driver, 12V větev |
| GPIO26 | LED pásek W1 (→ BC547B) | PWM | Přímé spínání, 5V |
| GPIO32 | LED pásek W2 (→ BC547B) | PWM | Přímé spínání, 5V |
| GPIO33 | Pasivní bzučák | PWM | Různé frekvence = různé tóny |
| GPIO18 | Servo SM-S2309S (signal) | PWM | 50 Hz, 1–2 ms pulz |
| GPIO19 | 74HC595N (DS — data) | Digitální | Shift register pro LCD |
| GPIO23 | 74HC595N (SH_CP — clock) | Digitální | Shift register pro LCD |
| GPIO15 | 74HC595N (ST_CP — latch) | Digitální | Shift register pro LCD |

---

## 🔧 Zapojení jednotlivých komponent

### 1. DHT22 — Teplota a vlhkost

```
         DHT22
        ┌───────┐
  3.3V ─┤ VCC   │
         │       │
  GPIO4 ─┤ DATA  │──┐
         │       │  │
   GND ─┤ GND   │  4.7kΩ
        └───────┘  │
                   3.3V (pull-up)
```

- VCC → 3.3V
- DATA → GPIO4
- GND → GND
- **4.7kΩ rezistor** mezi DATA a 3.3V (pull-up, nutný pro spolehlivou komunikaci)

---

### 2. KY-025 Reed Switch modul — Detekce otevření

```
  KY-025 modul
  ┌────────────┐
  │ A0  (nevyužito)
  │ G   ──→ GND
  │ +   ──→ 3.3V
  │ D0  ──→ GPIO13
  └────────────┘
```

- Trimmer na modulu slouží k nastavení citlivosti
- Magnet na víko krabice, modul na tělo
- Otevření → D0 změní stav → alarm

---

### 3. Flame sensor modul — Detekce ohně

```
  Flame sensor modul
  ┌────────────┐
  │ A0  (nevyužito)
  │ G   ──→ GND
  │ +   ──→ 3.3V
  │ D0  ──→ GPIO14
  └────────────┘
```

- Trimmer nastaví citlivost detekce
- Zapojení identické s reed switchem

---

### 4. Hg rtuťový nakláněcí spínač — Detekce převrácení

```
  Hg modul (3 piny)
  ┌────────────┐
  │ -   ──→ GND
  │ +   ──→ 3.3V
  │ S   ──→ GPIO27
  └────────────┘
```

---

### 5. Light blocking modul — Měření světla

```
  Light blocking modul (3 piny)
  ┌────────────┐
  │ -   ──→ GND
  │ +   ──→ 3.3V
  │ S   ──→ GPIO34
  └────────────┘
```

- ⚠️ GPIO34 je input-only pin (nemá interní pull-up)
- Modul má vlastní pull-up, takže to nevadí

---

### 6. Push button modul — Manual Override

```
  Button modul (3 piny)
  ┌────────────┐
  │ -   ──→ GND
  │ +   ──→ 3.3V
  │ S   ──→ GPIO35
  └────────────┘
```

- ⚠️ GPIO35 je input-only pin
- Modul má vlastní pull-up

---

### 7. Rotary encoder modul — Nastavení prahu

```
  Rotary encoder (5 pinů)
  ┌────────────┐
  │ GND ──→ GND
  │ +   ──→ 3.3V
  │ SW  ──→ GPIO5   (stisk)
  │ DT  ──→ GPIO17  (směr)
  │ CLK ──→ GPIO16  (takt)
  └────────────┘
```

- Otáčení mění teplotní práh ventilátoru
- Stisk (SW) může sloužit jako potvrzení / přepnutí režimu
- V ESPHome existuje nativní podpora `rotary_encoder`

---

### 8. Ventilátor 120mm — BC547B gate driver + IRF520

Toto je nejsložitější zapojení v projektu. BC547B zesiluje 3.3V signál z ESP32 na 12V pro gate IRF520.

```
                          +12V
                           │
                       ┌───┴───┐
                       │ 10kΩ  │ (pull-up)
                       └───┬───┘
                           │
  GPIO25 ──[1kΩ]──→ B ┌───┴──→ G ┌──────┐
                       │BC547B│    │IRF520│
                   E ──┘      S ──┘      D
                   │          │          │
                  GND        GND     Fan GND (-)
                                         │
                                    Fan VCC (+) ──→ +12V
```

**Podrobně:**
1. ESP32 GPIO25 → 1kΩ rezistor → BC547B **báze (B)**
2. BC547B **emitor (E)** → GND
3. BC547B **kolektor (C)** → IRF520 **gate (G)**
4. 10kΩ rezistor mezi IRF520 **gate (G)** a **+12V** (pull-up)
5. IRF520 **source (S)** → GND
6. IRF520 **drain (D)** → ventilátor GND (−)
7. Ventilátor VCC (+) → +12V

**Jak to funguje:** ESP32 pošle PWM na GPIO25 → BC547B sepne → stáhne gate IRF520 k zemi... ne, moment:

**Správná logika (invertovaná):**
- GPIO HIGH → BC547B sepne → gate IRF520 se stáhne na GND → MOSFET **vypne**
- GPIO LOW → BC547B rozepne → pull-up vytáhne gate na 12V → MOSFET **sepne**

⚠️ **Logika je invertovaná!** V ESPHome to vyřešíš jednoduše: `inverted: true` v konfiguraci PWM výstupu.

---

### 9. LED pásek Dual White (CCT) — BC547B přímé spínání

Dva nezávislé kanály, každý přes vlastní BC547B. Odběr ~3 mA/kanál.

```
  Kanál W1:                          Kanál W2:

       +5V                                +5V
        │                                  │
   LED pásek +                        LED pásek +
   LED pásek W1-                      LED pásek W2-
        │                                  │
        C                                  C
  GPIO26 ──[1kΩ]──→ B │BC547B│    GPIO32 ──[1kΩ]──→ B │BC547B│
                       E                               E
                       │                               │
                      GND                             GND
```

**Podrobně (pro každý kanál):**
1. LED pásek **+** → 5V
2. LED pásek **W1−** (nebo **W2−**) → BC547B **kolektor (C)**
3. ESP32 GPIO → 1kΩ rezistor → BC547B **báze (B)**
4. BC547B **emitor (E)** → GND

**Logika:** GPIO HIGH → BC547B sepne → proud teče přes LED → svítí. Přímá logika, žádná inverze.

---

### 10. Pasivní bzučák

```
  GPIO33 ──[100Ω]──→ Buzzer (+)
                      Buzzer (−) ──→ GND
```

- 100Ω rezistor omezuje proud
- PWM frekvence určuje tón (např. 1000 Hz = pípnutí, 500 Hz = varovný tón, 2000 Hz = alarm)
- V ESPHome se řídí přes `rtttl` buzzer platformu nebo `output` + `ledc`

---

### 11. Servo motor SM-S2309S — Mechanická klapka

```
  Servo (3 vodiče)
  ┌────────────┐
  │ GND (hnědý/černý)  ──→ GND
  │ VCC (červený)       ──→ 5V
  │ Signal (oranžový)   ──→ GPIO18
  └────────────┘
```

- PWM 50 Hz, pulz 1 ms (0°) až 2 ms (180°)
- V ESPHome: platforma `servo`
- ⚠️ Servo může při zátěži táhnout i 500+ mA — pokud se ESP32 resetuje, servo napájej ze samostatného 5V zdroje (ne přes ESP32 VIN)

---

### 12. LCD displej LCM1602C — přes 74HC595N shift register

Shift register 74HC595N převádí 3 GPIO piny na 6+ výstupů pro LCD. Šetří piny a vyhýbá se boot-citlivým GPIO.

```
  ESP32                  74HC595N                      LCD (LCM1602C)
  ┌──────┐              ┌──────────────┐              ┌──────────────┐
  │GPIO19├──→ pin 14 DS │              │              │              │
  │GPIO23├──→ pin 11 SH │  Q0 (pin 15) ├──→───────────┤ RS           │
  │GPIO15├──→ pin 12 ST │  Q1 (pin 1)  ├──→───────────┤ E            │
  │      │              │  Q2 (pin 2)  ├──→───────────┤ D4           │
  │ 3.3V ├──→ pin 16 VCC│  Q3 (pin 3)  ├──→───────────┤ D5           │
  │ 3.3V ├──→ pin 10 MR │  Q4 (pin 4)  ├──→───────────┤ D6           │
  │  GND ├──→ pin 8 GND │  Q5 (pin 5)  ├──→───────────┤ D7           │
  │  GND ├──→ pin 13 OE │              │              │              │
  └──────┘              └──────────────┘              │ VSS ──→ GND  │
                                                      │ VDD ──→ 5V   │
                                                      │ RW  ──→ GND  │
                                                      │ V0  ──→ Bochen│
                                                      │ A   ──→ 5V(220Ω)
                                                      │ K   ──→ GND  │
                                                      └──────────────┘
```

**74HC595N zapojení (DIP-16 pouzdro):**

| Pin | Název | Připojení |
|-----|-------|-----------|
| 8 | GND | GND |
| 10 | MR (master reset) | 3.3V (trvale HIGH = nikdy neresetovat) |
| 11 | SH_CP (shift clock) | ESP32 GPIO23 |
| 12 | ST_CP (latch clock) | ESP32 GPIO15 |
| 13 | OE (output enable) | GND (trvale LOW = výstupy vždy aktivní) |
| 14 | DS (serial data) | ESP32 GPIO19 |
| 15 | Q0 | LCD RS |
| 16 | VCC | 3.3V |
| 1 | Q1 | LCD E |
| 2 | Q2 | LCD D4 |
| 3 | Q3 | LCD D5 |
| 4 | Q4 | LCD D6 |
| 5 | Q5 | LCD D7 |
| 6–7 | Q6, Q7 | Nevyužito |
| 9 | Q7' | Nevyužito (sériový výstup pro řetězení) |

**LCD piny:**

| LCD pin | Připojení |
|---------|-----------|
| VSS | GND |
| VDD | 5V |
| V0 | Bochen 3296 střední pin (kontrast) |
| RS | 74HC595N Q0 |
| RW | GND |
| E | 74HC595N Q1 |
| D4 | 74HC595N Q2 |
| D5 | 74HC595N Q3 |
| D6 | 74HC595N Q4 |
| D7 | 74HC595N Q5 |
| A (backlight) | 5V přes 220Ω |
| K (backlight) | GND |

**Bochen 3296 potenciometr** (kontrast LCD):
```
  5V  ──→ pin 1 (krajní)
  GND ──→ pin 3 (krajní)
  pin 2 (střední / wiper) ──→ LCD V0
```

**Napětí:**
- 74HC595N napájený z **3.3V** — rozpozná 3.3V signály z ESP32
- LCD napájený z **5V** — ale přijímá 3.3V logiku (HD44780 VIH min = 2.2V)
- Toto zapojení funguje spolehlivě bez převodníků úrovní

⚠️ V ESPHome se 74HC595N konfiguruje přes platformu `sn74hc595` — nativní podpora, žádné externí knihovny

---

## 📋 Seznam potřebných rezistorů

| Hodnota | Počet | Použití |
|---------|-------|---------|
| 1kΩ | 3× | Báze BC547B (fan, LED W1, LED W2) |
| 10kΩ | 1× | Pull-up gate IRF520 na 12V |
| 4.7kΩ | 1× | Pull-up DHT22 DATA na 3.3V |
| 100Ω | 1× | Omezení proudu bzučáku |
| 220Ω | 1× | Backlight LCD displeje |

---

## ⚠️ Důležitá pravidla

1. **Společná zem (GND)** — GND z ESP32 (USB), GND z 12V adaptéru a GND všech komponent musí být propojené na jednom bodě na breadboardu.

2. **Nikdy nepřipojuj 12V na ESP32 GPIO** — GPIO piny tolerují max 3.3V. 12V = smrt čipu.

3. **Nikdy nenapájej ventilátor z ESP32** — ventilátor žere stovky mA, ESP32 to nedá. Vždy přes oddělený 12V zdroj a MOSFET.

4. **Servo pozor** — pokud se ESP32 při pohybu serva resetuje, servo potřebuje vlastní 5V napájení (ne přes ESP32).

5. **IRF520 bez gate driveru nefunguje s ESP32** — proto ten BC547B. Nikdy nezapojuj IRF520 gate přímo na ESP32 GPIO.

6. **Kontroluj polaritu** — před zapnutím vždy zkontroluj že + je na + a − na −. Obráceně zapojený elektrolytický kondenzátor může explodovat (naštěstí žádné nemáš).

7. **Nejdřív zapoj, pak zapni** — nikdy nepřepojuj vodiče pod napětím.

---

## 🧪 Postup oživování (doporučený)

1. **Osaď ESP32** na breadboard, připoj USB → zkontroluj že bootuje
2. **Připoj DHT22** → ověř čtení v ESPHome logu
3. **Připoj reed switch, flame, Hg, light blocking, button** → ověř binární senzory
4. **Připoj rotary encoder** → ověř změnu hodnoty
5. **Připoj pasivní bzučák** → otestuj tón
6. **Připoj servo** → otestuj pohyb
7. **Zapoj 12V větev** (adaptér → DC jack → breadboard) → propoj GND
8. **Sestav gate driver** (BC547B + IRF520 + rezistory) → připoj ventilátor
9. **Připoj LED pásek** přes BC547B → otestuj oba kanály
10. **LCD displej** jako poslední (nejsložitější zapojení)

---
