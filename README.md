# NOVOTEC 3-LP-0200021-2 — RS-232 Adapter (Reverse-Engineered)

![License](https://img.shields.io/github/license/feiglein74/novotec-rs232-adapter)
![Interface](https://img.shields.io/badge/interface-RS--232%20%2F%20TTL-555)

Reverse-engineered circuit analysis and pinout of the **NOVOTEC (PCB marking: NOVTEC)
3-LP-0200021-2** board: an opto-buffered, full-duplex TTL ↔ RS-232 adapter, most likely
from a fuel-station outdoor payment terminal (OPT) talking to a fuel pump controller.
Built around MAX232, 2× HCPL-2531 and 74HC367.

> Detailed documentation is in German.

## Key findings

- **Data lines work:** TXD and RXD were verified in both directions.
- **Handshake lines are dead on the RS-232 side.** They are wired on the board, but the EMI
  filters at connector pins 4 and 5 are not fitted, so the signals never reach the connector.
- **The MCU always sees handshake-in as asserted.** A 14.8 kΩ bias resistor from V+ to MAX232 R2IN holds it there.
- **Pin 6 is a constant ≈ +9.7 V** ("fake asserted" DSR/DCD towards the remote device).
- **No galvanic isolation.** The optocouplers buffer transients, but both sides share ground.
- **Supply:** 9–12 V DC into a 7805, giving 5 V logic and ±9.7 V from the MAX232 charge pump.

### 6-pin connector (RS-232 side)

| Pin | Function | Idle level |
|---|---|---|
| 1 | GND | 0 V |
| 2 | RXD in → MAX232 R1IN | — |
| 3 | TXD out ← MAX232 T1OUT | −9.7 V (mark) |
| 4 | handshake in — EMI filter not fitted | — |
| 5 | handshake out — EMI filter not fitted | — |
| 6 | constant +9.7 V ("always ready") | +9.7 V |

Signal paths, the TTL header, the test log and open questions are covered below (German).

---

# RS-232-Adapter — Schaltungsanalyse

## Identifikation

- **Platinenbeschriftung:** `NOVTEC 3-LP-0200021-2`
- **Hersteller:** NOVOTEC Computer-Systeme GmbH, Karlsruhe (gegründet 1988)
- **Webseite:** [novotec.org](https://www.novotec.org/en/company/company-profile.html)
- **Bedeutung der Nummern (Hypothese):**
  - `3-LP` = Leiterplatten-Kennung (Produkt-Familie / Layouter-Code)
  - `0200021` = laufende Artikelnummer
  - `-2` = Revision / Version 2

## Anwendungskontext (wahrscheinlich)

NOVOTEC Computer-Systeme GmbH baut **Production Data Acquisition Systeme** mit
Hauptprodukt **Outdoor Payment Terminals (OTPs) für Tankstellen und
Flottenbetriebe** (Auto-Tankstellen-Bezahlsysteme).

Die Schaltung passt funktional sehr gut zu einer **Schnittstellen-Karte
zwischen einem Bezahlterminal und einer Zapfsäule** (oder ähnlichem
RS-232-fähigen Außengerät):

- **Zapfsäulen-Protokolle laufen klassisch auf RS-232** (oft 9600/4800 Baud,
  proprietäre Telegramm-Formate).
- **Elektrisch dreckige Umgebung**: Pumpen, Magnetventile, Außenanlage mit
  Erdpotenzial-Unterschieden — opto-gepufferte und EMI-gefilterte RS-232
  ist hier sinnvoll.
- **Permanent assertiertes Handshake-In am MCU** passt zur Erwartung der
  Zapfsäulen-Steuerung, dass das Terminal dauerhaft „bereit" ist.
- **Unbestückte EMI-Filter auf den Handshake-Lines** sind dann eine
  bewusste Konfigurations-Bestückung: für diese Anwendung wird kein
  echter Hardware-Handshake gebraucht, nur die Datenkanäle.

Vermutliche Epoche: **1990er bis frühe 2000er**, basierend auf der DIP-Bestückung
und der MAX232ACPE-Variante.

**Für Datenblatt / Schaltplan:** Anfrage bei NOVOTEC mit der Nummer
`3-LP-0200021-2` über die Webseite.

## Funktion

Opto-gepufferter **TTL ↔ RS-232 Vollduplex-Adapter**. Verbindet einen Mikrocontroller
oder Logikbaustein (TTL-Pegel am 2×5-Pfostenstecker) mit einem Endgerät auf der
RS-232-Seite (am 6-pin-Stecker).

- **Datenkanäle (TXD/RXD):** funktional, beide Richtungen verifiziert.
- **Handshake-Kanäle:** intern vollständig verdrahtet, **extern auf 6-pin-Seite
  durch unbestückte EMI-Filter physikalisch unterbrochen**.
- **Handshake-In am MCU:** wird durch eine Bias-Schaltung am MAX232 R2IN
  permanent auf „asserted" gehalten — die MCU-Software bekommt also dauerhaft
  „bereit"-Signal und hängt nicht auf einem Handshake-Wait.
- **6-pin Pin 6:** liefert dauerhaft ≈+9,7 V als „fake asserted DSR/DCD"
  Richtung Endgerät.

## Komponenten

| Bauteil | Funktion |
|---|---|
| 7805 + Kondensatoren | 5 V-Linearregler aus 9–12 V Versorgung |
| 74HC367N | Hex Tri-State Buffer (4+2 Kanäle), beide OE auf GND → dauerhaft enabled |
| 2× HCPL-2531 | Dual-Highspeed-Optokoppler — Signalpufferung |
| MAX232ACPE | TTL ↔ RS-232 Pegelwandler mit Ladungspumpe (±10 V aus 5 V) |
| L51S103 | Widerstandsnetzwerk 10 kΩ — Pull-ups am 2×5-Eingang |
| 14,8 kΩ Widerstand | R2IN-Bias-Schaltung (fake asserted Handshake-In) |
| 1× 3-pin EMI-Filter (braunes Plättchen) | bestückt nur an 6-pin Pin 6 |
| 2× 3-pin EMI-Filter | fehlen an 6-pin Pin 4 und Pin 5 (Handshake-Pins) |

## Versorgung

- **7805 Eingang:** 9–12 V DC (Quelle noch nicht abschließend geklärt — vermutlich
  über 6-pin selbst oder separat).
- **7805 Ausgang:** 5 V → versorgt 74HC367, beide 2531, MAX232.
- **MAX232 V+ (Pin 2):** ≈+9,7 V (Ladungspumpe).
- **MAX232 V− (Pin 6):** ≈−9,7 V (Ladungspumpe).

**Wichtig:** Trotz Optokoppler-Topologie ist die **Masse gemeinsam** zwischen
MCU-Seite und RS-232-Seite — **keine galvanische Trennung**. Die Optos
puffern Spikes und Transienten, isolieren aber nicht.

## Anschlussbelegung

### 2×5-Pfostenstecker (TTL-Seite, zum MCU)

10 kΩ Pull-ups (Widerstandsnetzwerk L51S103) auf den Signalleitungen.

| Funktion | über 74HC367-Pin |
|---|---|
| TXD vom MCU (Eingang) | 367 Pin 14 (2A2) |
| Handshake-Out vom MCU (Eingang) | 367 Pin 12 (2A1) |
| RXD zum MCU (Ausgang) | 367 Pin 5 (1Y2) |
| Handshake-In zum MCU (Ausgang) | 367 Pin 7 (1Y3) |
| VCC, GND, weitere | restliche Pins (Belegung noch zu klären) |

### 6-pin-Anschluss (RS-232-Seite, zur Maschine)

| Pin | Funktion | Status | Pegel idle |
|---|---|---|---|
| 1 | GND | ✓ | 0 V |
| 2 | RXD rein → MAX232 R1IN (Pin 13) | ✓ funktional | 0 V (offen) |
| 3 | TXD raus ← MAX232 T1OUT (Pin 14) | ✓ funktional | −9,7 V (MARK) |
| 4 | wäre Handshake — EMI-Filter fehlt | ✗ tot | — |
| 5 | wäre Handshake-Out (T2OUT) — EMI-Filter fehlt | ✗ tot | — |
| 6 | fake asserted (≈+9,7 V via V+) | „permanent ready" | +9,7 V |

## Signalpfad — TX (MCU → Maschine)

```
2×5 (TXD) ─► 367 Pin 14 (2A2) ─► 367 Pin 13 (2Y2) ─► 2531 Pin 3 (LED2-K)
                                                          │
                                                       LED 2 ── Pin 4 (Anode) ── R ── VCC
                                                          │
                                                       Opto-Barriere
                                                          │
                                                     2531 Pin 6 (Vo2) ─► MAX232 Pin 11 (T1IN)
                                                                              │
                                                                          (intern: TTL → RS-232)
                                                                              │
                                                                       MAX232 Pin 14 (T1OUT) ─► 6-pin Pin 3
                                                                                                (RS-232 ±9,7 V)
```

**Verifiziert:** 367 Pin 14 auf GND ziehen → 6-pin Pin 3 springt von −9 V auf +9 V.

## Signalpfad — RX (Maschine → MCU)

```
6-pin Pin 2 ─► MAX232 Pin 13 (R1IN, ±9,7 V RS-232)
                  │
              (intern: RS-232 → TTL)
                  │
              MAX232 Pin 12 (R1OUT, TTL) ─► 2. 2531 LED-Seite
                                                  │
                                              Opto-Barriere
                                                  │
                                              2. 2531 Vo ─► 367 Pin 4 (1A2) ─► 367 Pin 5 (1Y2) ─► 2×5 (RXD)
```

**Verifiziert:** 6-pin Pin 2 zwischen V+ und V− toggeln → 2×5-RXD-Pin toggelt zwischen LOW und HIGH.

## Handshake-Schaltung (intern aktiv, extern tot)

### Handshake-In (Maschine → MCU)

```
MAX232 V+ (Pin 2, +9,7 V) ─── 14,8 kΩ ──┬── MAX232 Pin 8 (R2IN)
                                        │
                                  (chip-intern 5 kΩ → GND)

   Spannungsteiler:
   V_Pin8 = 9,7 V × 5 k / (14,8 k + 5 k) = 2,58 V  → über Threshold (≈+1,4 V)

   → MAX232 interpretiert „SPACE / asserted"
   → R2OUT (Pin 9) → TTL LOW
   → treibt 2. 2531 Pin 3 (LED-Kathode) → LED an → Opto-Output LOW
   → 367 (1A3, Pin 6) → 367 Pin 7 (1Y3) LOW → 2×5 RX-Handshake
   → MCU sieht permanent „Handshake-In = asserted"
```

### Handshake-Out (MCU → Maschine)

```
2×5 (Hsh-Out) ─► 367 Pin 12 (2A1) ─► 367 Pin 11 (2Y1) ─► 1. 2531 Pin 2 (LED1-K)
                                                              │
                                                           Opto-Barriere
                                                              │
                                                         2531 Pin 7 (Vo1) ─► MAX232 Pin 10 (T2IN)
                                                                                  │
                                                                              (intern → RS-232)
                                                                                  │
                                                                            MAX232 Pin 7 (T2OUT)
                                                                                  │
                                                                            ✗ EMI-Filter Pin 5 fehlt
                                                                                  │
                                                                                  ↓
                                                                              [offen, endet hier]
```

## Ein-/Ausgangsübersicht

```
                            ┌──────────────────────┐
        9–12 V DC ─►  7805 ─►  5 V intern (VCC)    │
                            │  ±9,7 V via MAX232    │
                            │  Ladungspumpe          │
                            └──────────────────────┘

   ┌─────────────────────────────┐         ┌─────────────────────────────┐
   │  2×5 Pfostenstecker (TTL)   │         │   6-pin Stecker (RS-232)    │
   │                              │         │                              │
   │ TXD MCU  ─► [367 → 2531] ───►│ ──TTL──►│ ──► MAX232 T1 ──► Pin 3 ─►   │ TXD (raus, −9,7 V idle)
   │                              │         │                              │
   │ RXD MCU  ◄── [367 ◄─ 2531] ──│ ◄──TTL──│ ◄── MAX232 R1 ◄── Pin 2 ◄─   │ RXD (rein)
   │                              │         │                              │
   │ Hsh-Out  ─► [367 → 2531] ───►│ ──TTL──►│ ──► MAX232 T2 ──► Pin 5 ✗   │ EMI-Filter fehlt
   │                              │         │                              │
   │ Hsh-In   ◄── [367 ◄─ 2531] ──│ ◄──TTL──│ ◄── MAX232 R2 ◄── Pin 4 ✗   │ EMI-Filter fehlt
   │                              │         │                              │
   │ (R-Netz. 10k Pull-ups)       │         │ Pin 6: +9,7 V (fake asserted)│
   │                              │         │ Pin 1: GND                   │
   └─────────────────────────────┘         └─────────────────────────────┘
```

## Offene Punkte

1. **6-pin Pin 6 Topologie:** wie genau hängt Pin 6 an V+ (Pin 2 MAX232)? Direkter
   Trace oder über zusätzlichen Widerstand? Bisher verifiziert: 9,7 V steady, Cap-Kette
   (3-pin-Filter → 10 kΩ → 1 µF) führt zur VCC-Schiene (Pin 16), DC-blockierend
   wirksam — die +9,7 V müssen über einen anderen Pfad eingespeist werden.
2. **7805 Eingangs-Pfad VIN:** Quelle der 9–12 V noch nicht abschließend identifiziert.
3. **2×5 Pin-Layout:** welche physische Pin-Nummer am 2×5 ist welche Funktion?
   Die 4 Signal-Pins gehen zu 367 Pin 5, 7, 12, 14 — die übrigen 6 Pins (VCC, GND,
   weitere?) noch zu klären.
4. **Handshake-Konvention:** RTS/CTS oder DTR/DSR — lässt sich aus der Schaltung
   allein nicht ableiten, hängt vom Originalgerät ab. Da die Lines ohnehin tot
   sind, eher akademisch.

## Testprotokoll

### Statischer Test (verifiziert ✓)

| Aktion | Erwartung | Beobachtet |
|---|---|---|
| 367 Pin 14 auf GND ziehen | 6-pin Pin 3 von −9 V auf +9 V | ✓ getoggelt |
| 367 Pin 14 loslassen | 6-pin Pin 3 zurück auf −9 V | ✓ |
| 6-pin Pin 2 auf V+/V− toggeln | 2×5-RXD-Pin LOW/HIGH | ✓ getoggelt |

→ **Beide Signal-Strecken funktional.**

### PC-Schleifentest (vorgesehen)

1. Versorgung 9–12 V DC anlegen.
2. Auf 2×5-Seite: TXD (Draht zu 367 Pin 14) und RXD (Draht von 367 Pin 5) **direkt verbinden**.
3. Auf 6-pin: USB-RS232-Adapter anschließen:
   - 6-pin Pin 1 ← PC GND
   - 6-pin Pin 2 ← PC TXD
   - 6-pin Pin 3 → PC RXD
4. Terminal: **9600 Baud, 8N1, kein Flow-Control, local echo OFF**.
5. Tasten tippen → Echo erwartet.

Falls die Baudrate des Endgeräts nicht 9600 ist: bei Schleifentest unkritisch
(Sender = Empfänger = PC). Nur müssen Sende- und Empfangsrate am PC identisch
eingestellt sein.

## Diagnose

> **Funktionsfähiger Vollduplex-TTL↔RS-232-Adapter mit Opto-Pufferung.
> Handshake-Lines durch fehlende EMI-Filter deaktiviert; Bias-Schaltung
> simuliert dauerhaft asserted Handshake-In für die MCU.**

Typische Anwendung: kleines MCU-Board steuert über RS-232 ein Industrie-,
Mess- oder Werkstatt-Gerät, das mit Datenkanal allein auskommt aber CTS/DSR
„assertiert" sehen will, damit die Kommunikation nicht hängenbleibt.
