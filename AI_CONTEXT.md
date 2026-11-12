# Projekt-Briefing: Dynakku – Studienarbeit DHBW Karlsruhe

## 1. Rahmendaten & Aufgabenstellung
- **Thema:** USB-Powerbank zur Ladung über Fahrraddynamo an Biorädern (Nachrüstlösung).
- **Autor:** Loris De Marco (Kurs TEI24B1, Matrikelnr. 9553410).
- **Betreuer:** Joachim Manz (DHBW).
- **Ziel:** Pufferung von Dynamoenergie in einem Akku zur unterbrechungsfreien Versorgung von Verbrauchern via USB-C (10–15 W PD).

## 2. Generator-Parameter (Nabendynamo)
Aus Messungen (Kelvin-Messung, Leerlauf- & Kurzschlussversuch mit 0,3-Ω-Shunt) ermitteltes lineares Modell des Shimano DH-T4050-1D:
- **Polpaarzahl:** p = 14 (elektrische Frequenz f = 1,784 · v in km/h)
- **Wicklungswiderstand:** R_i = 3,2 Ω (Vierleitermessung: U_K = 1,6 V bei I_C = 0,5 A)
- **Synchrone Induktivität (effektiv):** L_s = 116,7 mH
- **Leerlaufspannungskonstante:** k_v = 0,59 V_RMS / (km/h) bzw. V_peak = sqrt(2) · 0,59 · v ≈ 0,834 · v
- **Referenzlauf (Normgeschwindigkeit bei 15 km/h):**
  - Elektrische Frequenz: f = 26 Hz (theoretisch ~26,8 Hz)
  - Leerlaufspannung: V_RMS ≈ 8,78 V (bzw. ~8,8 V), V_peak ≈ 12,4 V
  - Kurzschlussstrom: I_k = I_S ≈ 453 mA (über 0,3-Ω-Shunt mit U_S = 136 mV)
  - Serienkompensationskapazität: C_s,theor ≈ 321,1 µF (praktisch C_s = 330 µF gewählt)

## 3. Schaltungsarchitektur & Baugruppen
1. **Eingangsstufe & Serienkompensation:**
   - **Anschlüsse:** J1 (AC+), J2 (AC-) als 2,54-mm-Lötpins.
   - **Kompensationskapazität (C1):** 330 µF / 50 V bipolarer Elektrolytkondensator (Panasonic ECE-A1HN331U, THT radial) in Reihe zu J1 (Resonanzpunkt bei ~15 km/h hebt L_s auf).
   - **Gleichrichterbrücke (D1–D4):** 4x SS34 Schottky-Dioden (40 V / 3 A, DO-214AB / SMC).
   - **Trenndiode (D6):** 1x SS34 Schottky-Diode (DO-214AB / SMC) zwischen Brückenausgang und Zwischenkreis (VDC), verhindert Rückentladung von C2 in den OVP-Kurzschließer.
2. **Überspannungsschutz (OVP):**
   - **Kurzschließer-MOSFET (Q1):** TI CSD17577Q3A (N-Kanal Power-MOSFET, 30 V, VSONP-8), schließt den Gleichrichterausgang vor D6 gegen GND kurz, sobald VDC > ~21–22 V steigt.
   - **Ansteuerung:** 20-V-Zenerdiode D5 (BZX84C20, SOT-23) von VDC über Vorwiderstand R1 (2,2 kΩ, 0603) und Gate-Pulldown R2 (10 kΩ, 0603) an das Gate von Q1.
3. **Gleichstrom-Zwischenkreis:**
   - **Glättungselko (C2):** 1000 µF / 50 V SMD-Elektrolytkondensator (Panasonic EEV-FK1H102V bzw. Nichicon UUE1V102MNS6MS, Bauform 18x22 mm) zur Pufferung der doppelten Dynamofrequenz.
4. **MPPT-Laderegler (Synchroner Buck-Converter):**
   - **Lade-Controller (U1):** TI BQ24650RVAR (VQFN-16).
   - **Leistungsstufe:** Synchron-Halbbrücke aus High-Side-MOSFET Q2 und Low-Side-MOSFET Q3 (jeweils TI CSD17577Q3A, VSONP-8), Bootstrap-Diode D7 (SS34) mit Bootstrap-Kondensator C6 (100 nF / 50 V, 0603) sowie Speicherdrossel L1 (10 µH, Vishay IHLM2525CZER100M01).
   - **Eingangs- & Hilfsspannungsfilter:** Eingangs-MLCCs C8, C9, C10 (3x 1 µF / 50 V, 0805), VCC-Tiefpass aus R8 (10 Ω, 0603) und C5 (1 µF / 50 V, 0805), Stützkondensatoren für VREF (C3: 1 µF / 50 V, 0805) und REGN (C7: 1 µF / 50 V, 0805).
   - **MPPT-Konfiguration (MPPSET):** Spannungsteiler aus R3 (750 kΩ, 0603) und R4 (100 kΩ, 0603) mit Filterkondensator C4 (100 nF / 50 V, 0603) klemmt die minimale Zwischenkreisspannung auf V_MPP = 1,2 V · (1 + 750k/100k) = 10,2 V.
   - **Ladeschlussspannung (VFB):** Spannungsteiler aus R10 (100 kΩ, 0603) und R11 (100 kΩ, 0603) programmiert V_BAT,max = 2,1 V · (1 + 100k/100k) = 4,20 V.
   - **Ladestrommessung & Ausgang:** Kelvin-Messshunt R9 (20 mΩ, Panasonic ERJ-3BWFR020V, 0603) für I_charge = 40 mV / 20 mΩ = 2,0 A Maximalstrom, gefiltert mit C11 (100 nF / 50 V, 0603), sowie Ausgangs-MLCCs C12, C13 (2x 10 µF / 16 V, 0805).
   - **Temperaturüberwachung (TS):** Festwiderstandsnetzwerk aus R5 (10 kΩ an VREF) sowie R6 + R7 (je 10 kΩ an GND).
5. **Akkuzelle & Batterieschutzschaltung (BMS):**
   - **Akkuzelle:** 1S Li-Ion (Format 21700, ca. 5000 mAh / ~18,5 Wh), angebunden über Lötpins J4 (+BATT) und J5 (-BATT) sowie SMD-Schmelzsicherung F1 (8 A, Bourns SF-1206HH800R-2, 1206).
   - **Schutz-IC (U2):** TI BQ29700DSER (WSON-6) mit RC-Versorgungsfilter aus R12 (300 Ω, 0603) und C14 (100 nF / 50 V, 0603) sowie Strommess-Vorwiderstand R13 (2,2 kΩ, 0603) an Pin V-.
   - **Schaltorgane:** Back-to-Back N-Kanal-MOSFETs Q4 (COUT) und Q5 (DOUT) im Massepfad (jeweils TI CSD17577Q3A, VSONP-8).
6. **USB-C Ausgangsstufe (5 V / 3 A Boost):**
   - **Aufwärtswandler (U3):** TI TPS61022RWUR (VQFN-7, V_IN: 3,0–4,2 V, V_OUT: ~5,1 V bis 3 A / 15 W), aktiviert über Lötjumper JP1.
   - **Leistungs- & Filterbauteile:** Speicherdrossel L2 (1 µH, Vishay IHLP2020BZER1R0M01), Eingangs-MLCCs C15, C16 (2x 10 µF / 16 V, 0805), Ausgangs-MLCCs C17, C18 (2x 10 µF / 16 V, 0805).
   - **Spannungseinstellung (FB):** Spannungsteiler aus R14 (750 kΩ, 0603) und R15 (100 kΩ, 0603).
   - **USB-C-Schnittstelle:** Buchse J6 (Same Sky UJC-H-G-SMT-2-P6-TR, liegend, Power-only), ESD-Schutzdiode D8 (ESD5Z5.0T5G, SOD-523) an VBUS sowie Pull-up-Widerstände R16 und R17 (je 10 kΩ, 0603) an CC1/CC2 nach VBUS zur Signalisierung von 3 A im USB Type-C Current Mode (gemäß DIN EN IEC 62680-1-3).

## 4. Software- & Toolchain-Umgebung
- **Textsatz:** LaTeX (main.tex, biblatex mit biber, Vorlage angepasst auf DHBW/SEW-Kopfzeile).
- **Simulation:** LTspice (Ersatzschaltbild-Modelle für Dynamo, OVP, TPS61022).
- **PCB-Design:** KiCad 8 mit MCP-Anbindung (`uvx kicad-mcp-pro`) im Workspace.