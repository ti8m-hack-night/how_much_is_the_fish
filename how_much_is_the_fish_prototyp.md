# how_much_is_the_fish – UX-Prototyp

Strompreis Schweiz (CH): Verlauf, Statistiken und günstigstes Zeitfenster für eine Last (z. B. E-Auto laden).
Stundenwerte in ct/kWh, Zeiten in Europe/Zurich. Design: Oblique (Schweizer Bundesverwaltung).

Interaktiver Prototyp (Beispieldaten): https://claude.ai/artifact/NDk9jefzCNykxV8FLpTe11

## Design-Grundlagen

Quelle: `oblique-bit/oblique`, `_palette.scss`. Schrift: Noto Sans (Fallback Arial, system-ui).

| Rolle | Hell | Dunkel | Oblique-Token |
|---|---|---|---|
| Hintergrund | `#f0f4f7` | `#131b22` | secondary-50 / -900 |
| Fläche (Cards) | `#ffffff` | `#1c2834` | white / secondary-800 |
| Text | `#1c2834` | `#f0f4f7` | secondary-800 / -50 |
| Text gedämpft | `#46596b` | `#acb4bd` | secondary-500 / -200 |
| Primäraktion | `#46596b` | `#dfe4e9` | secondary-500 / -100 |
| Akzent (Top-Linie, Prototyp-Badge) | `#d8232a` | `#fc656b` | red-600 / -400 |
| Günstig (unter Schwelle, Minimum) | `#047857` | `#34d399` | green-700 / -400 |
| Teuer (Maximum, Fehler) | `#bf1f25` | `#fc656b` | red-700 / -400 |
| Zeitfenster | `#1e40af` | `#dbeafe` | blue-800 / -100 |
| Warnung (Offline-Stand) | `#c2410c` | `#fb923c` | orange-700 / -400 |
| Wetter | `#8655f6` | `#c4a8ff` | purple-500 |

Raster und Masse: 4 px Radius (Buttons 2 px), 5 px roter Strich am oberen Seitenrand, Abstände 4/8/12/16/24/32 px, Mindesthöhe Bedienelemente 36 px, maximale Breite 1240 px. Breakpoints 600 / 905 / 1240 px. Hell- und Dunkelmodus folgen dem System.

## Seitenaufbau (von oben nach unten)

1. **Kopf**: Titel mit Badge «Prototyp», Untertitel, Datenstatus-Pille.
2. **Hinweis**: «UX-Prototyp. Alle Preise und Wetterwerte sind Beispieldaten.»
3. **Auswahl** (eine Zeile, bricht um): Tag (Heute / Morgen), Datumsfeld für anderen Tag, Schwellwert-Regler mit Wert, Wetter (Temperatur / Sonnenschein / Aus).
4. **Preisverlauf**: Säulendiagramm mit Legende.
5. **Zwei Spalten** (auf Mobil untereinander): Statistik links, Zeitfenster-Rechner rechts.
6. **Fuss**: Quellenangabe «Energy-Charts / Fraunhofer ISE», «Open-Meteo», Schalter zum Simulieren eines API-Ausfalls.

## Komponenten

### Datenstatus (F-1, F-3)
- Live: grüner Punkt, «Live · aktualisiert 14:05 Uhr».
- Ausfall (503/Netz): orange Pille, «Energy-Charts nicht erreichbar (503). Gespeicherter Stand von vor 3 h 12 min.» Keine Fehlerseite, Diagramm und Rechner bleiben nutzbar.

### Preisverlauf (F-4 bis F-7)
- Eine Säule pro Stunde, y-Achse ct/kWh, Nulllinie sichtbar bei negativen Preisen.
- Anzahl Säulen 24, an Zeitumstellungstagen 23 oder 25 (Hinweis unter dem Diagramm).
- Günstigste Stunde grün, teuerste rot, jeweils mit Beschriftung «min 3.00 · 03:00» / «max 14.20 · 19:00».
- Schwellwert als grün gestrichelte Linie; Stunden darunter grün hervorgehoben mit Balken unter der Achse.
- Wetter auf zweiter Achse rechts (Linie, violett): Temperatur in °C oder Sonnenschein in min/h.
- Gefundenes Zeitfenster als blau hinterlegter Bereich mit Rahmen (F-15).
- Tooltip pro Säule: Uhrzeit und Preis.

### Statistik (F-9, F-10)
Sechs Kennzahlen im Raster: Durchschnitt, Standardabweichung, Minimum (mit Uhrzeit), Maximum (mit Uhrzeit), Stunden unter Schwellwert («7 von 24»), Vergleich zum Vortag (Differenz in ct/kWh und %, rot bei teurer, grün bei günstiger).

### Zeitfenster-Rechner (F-12 bis F-14, F-16)
- Eingaben: Leistung P (kW), Dauer T (ganze Stunden), Von / Bis (Datum und Stunde, innerhalb der verfügbaren Daten).
- Ausgabe in blauer Ergebnisbox: Start und Ende, Kosten (EUR), Kosten bei Durchschnittspreis, Ersparnis.
- Zusatzzeile: Vergleich mit den T günstigsten Einzelstunden (nicht zusammenhängend).
- Fehlerfall (A-4): roter Hinweis «Im gewählten Zeitraum sind nur 6 Stunden verfügbar, gewünscht sind 8. Verkürze die Dauer oder erweitere den Zeitraum.»

## Berechnung (ohne API-Zugriff aufrufbar)

- **Aggregation**: Zeitreihe aus `unix_seconds` (UTC) und `price` (EUR/MWh) in 15-Minuten- oder 1-Stunden-Auflösung, Mittelwert je Stunde.
- **Umrechnung**: ct/kWh = EUR/MWh ÷ 10.
- **Statistik**: Mittelwert, Minimum, Maximum, Standardabweichung (Grundgesamtheit).
- **Fenstersuche**: gleitende Summe über T Stunden. Kosten = Σ (Preis in EUR/kWh) × P × 1 h.
- **Zeitzone**: Darstellung und Tageszuordnung über `Europe/Zurich`.

### Abnahmebeispiel
Preise 00:00 bis 06:00 in EUR/MWh: 80, 60, 40, 30, 35, 70. P = 11 kW, T = 3 h.

| Grösse | Erwartung |
|---|---|
| Günstigstes Fenster | 02:00–05:00 |
| Kosten im Fenster | 1.155 EUR |
| Kosten bei Durchschnittspreis (52.5 EUR/MWh) | 1.7325 EUR |
| Ersparnis | 0.5775 EUR |
| Durchschnitt / Minimum / Maximum | 52.5 / 30 (03:00) / 80 (00:00) EUR/MWh |
| Umrechnung 52.5 EUR/MWh | 5.25 ct/kWh |

## Abdeckung der Anforderungen

| Priorität | Im Prototyp gezeigt |
|---|---|
| Muss | F-1 bis F-6, F-9, F-12 bis F-14 |
| Soll | F-7 (Wetter), F-10 (Vortag), F-15 (Fenster im Diagramm) |
| Kann | F-16 (Einzelstunden-Vergleich). F-8 (7 Tage) und F-11 (Korrelation) fehlen. |

## Hinweise für die lokale Umsetzung

- Die Preise werden live geladen und lokal zwischengespeichert; bei Fehlern zeigt die App den letzten Stand mit Alter.
- Die Beispieldaten im Prototyp ersetzt der Abruf von `https://api.energy-charts.info/price?bzn=CH` und Open-Meteo.
- Quelle bei Veröffentlichung nennen: «Energy-Charts / Fraunhofer ISE» (nichtkommerzielle Nutzung).
