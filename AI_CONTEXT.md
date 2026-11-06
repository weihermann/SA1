# Projekt-Briefing: Dynakku – Studienarbeit DHBW Karlsruhe

## 1. Rahmendaten & Aufgabenstellung
- **Thema:** USB-Powerbank zur Ladung über Fahrraddynamo an Biorädern (Nachrüstlösung).
- **Autor:** Loris De Marco (Kurs TEI24B1, Matrikelnr. 9553410).
- **Betreuer:** Joachim Manz (DHBW).
- **Ziel:** Pufferung von Dynamoenergie in einem Akku zur unterbrechungsfreien Versorgung von Verbrauchern via USB-C (10–15 W PD).

## 2. Generator-Parameter (Nabendynamo)
Aus Messungen (Kelvin-Messung, Leerlauf- & Kurzschlussversuch mit 0,3-Ω-Shunt) ermitteltes lineares Modell:
- **Polpaarzahl:** p = 14 (elektrische Frequenz f = 1,784 · v in km/h)
- **Wicklungswiderstand:** R_i = 3,25 Ω
- **Synchrone Induktivität (effektiv):** L_s = L_eff = 116,6 mH (strombegrenzende Ankerrückwirkung)
- **Leerlaufspannungskonstante:** k_v = 0,602 V_RMS / (km/h) bzw. V_peak = sqrt(2) · 0,602 · v
- **Referenzlauf:** Bei 20 km/h: f ≈ 35,7 Hz, V_peak ≈ 21,2 V, Kurzschlussstrom I_k ≈ 550 mA.

## 3. Schaltungsarchitektur & Baugruppen
1. **Eingangsstufe & Serienkompensation:**
   - Kompensationskapazität C_s = 330 µF (bipolar) in Reihe zur AC-Wicklung (Resonanzpunkt bei ~15–18 km/h hebt L_s auf).
   - Gleichrichterbrücke: 4x SS34 Schottky-Dioden (mind. 40 V / 3 A).
   - Trenndiode D_sep (SS34) zum Zwischenkreis.
2. **Überspannungsschutz (OVP):**
   - N-MOSFET (Q1) schließt V_RAW vor D_sep gegen Masse kurz, wenn die Zwischenkreisspannung V_DC > 20–22 V steigt.
   - Ansteuerung über 20-V-Zenerdiode mit 1-kΩ-Vorwiderstand und 10-kΩ-Gate-Pulldown.
3. **Gleichstrom-Zwischenkreis:**
   - Glättungselko C_ZK = 1000–2200 µF / 50 V Low-ESR (fängt die 15–35 Hz Halbkörperschwingungen ab).
4. **MPPT-Lader & Batteriemanagement:**
   - Lade-IC: TI BQ24650 (MPPSET eingestellt auf ca. 10,2 V Eingangsklemmung, Ladeschlussspannung 4,20 V).
   - Akkuzelle: 1S Li-Ion (Format 21700, ca. 5000 mAh / ~18,5 Wh).
   - Schutzschaltung: TI BQ29700 mit Back-to-Back N-MOSFETs (Q4/Q5).
   - Kelvin-Strommessung über Shunt R9 mit separater Rückführung an SRN (Pin 9).
5. **USB-Ausgangsstufe:**
   - Aufwärtswandler: TI TPS61022 (V_IN: 3,0–4,2 V vom Akku, V_OUT: 5,1 V stabil bis 3 A / 15 W).
   - Festwiderstände an CC1/CC2 zur USB-C Typ-C Signalisierung (3 A @ 5 V).

## 4. Software- & Toolchain-Umgebung
- **Textsatz:** LaTeX (main.tex, biblatex mit biber, Vorlage angepasst auf DHBW/SEW-Kopfzeile).
- **Simulation:** LTspice (Ersatzschaltbild-Modelle für Dynamo, OVP, TPS61022).
- **PCB-Design:** KiCad 8 mit MCP-Anbindung (`uvx kicad-mcp-pro`) im Workspace.