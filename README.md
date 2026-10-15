# how_much_is_the_fish

Wie viel kostet der Strom, und wann ist er am günstigsten? Strompreis-Verlauf, Statistiken und das günstigste Zeitfenster für eine Last (z. B. E-Auto laden).

> **UX-Prototyp:** _Link folgt (wird vor der Hack Night bereitgestellt)_
> **Zeitrahmen:** 1 Stunde. Priorisiert: zuerst alle **Muss**-Anforderungen, dann **Soll**, dann **Kann**.
> **Technologie, Git:** frei wählbar bzw. optional. Die Anwendung läuft **lokal**, die Daten werden aus dem **Internet** geladen.

## 1. Datenquellen (Pflicht)

**Strompreis:** [Energy-Charts API](https://api.energy-charts.info/) (Fraunhofer ISE), kein API-Key nötig.

```
GET https://api.energy-charts.info/price?bzn=CH
GET https://api.energy-charts.info/price?bzn=CH&start=2026-10-14&end=2026-10-15
```

Die Antwort ist JSON mit Zeitstempeln (`unix_seconds`, UTC) und Preisen (`price`) in **EUR/MWh**. Die Auflösung kann **15 Minuten oder 1 Stunde** betragen. Die App muss beides verarbeiten und zu Stundenwerten (Mittelwert) zusammenfassen. Umrechnung: **ct/kWh = EUR/MWh ÷ 10**.

**Wetter (Soll):** [Open-Meteo](https://open-meteo.com/en/docs), kein API-Key nötig.

```
GET https://api.open-meteo.com/v1/forecast?latitude=47.37&longitude=8.54&hourly=temperature_2m,sunshine_duration&timezone=Europe%2FZurich
```

**Hinweise:** Die Energy-Charts-API war in der Vergangenheit zeitweise nicht erreichbar (HTTP 503). Der lokale Cache (F-3) ist deshalb Pflicht. Bei Veröffentlichung ist "Energy-Charts / Fraunhofer ISE" als Quelle zu nennen (nichtkommerzielle Nutzung).

## 2. Anforderungen

### Daten laden
- **F-1 (Muss):** Preise werden live von der API geladen, für heute und, sobald verfügbar, für morgen. Es gibt keine fest einkompilierten Preisdaten.
- **F-2 (Muss):** Zeiten werden in der Zeitzone Europe/Zurich dargestellt (Sommer- und Winterzeit korrekt, auch an Umstellungstagen).
- **F-3 (Muss):** Antworten werden lokal zwischengespeichert. Bei Ladefehlern (503, Netz) zeigt die App den letzten gespeicherten Stand mit Hinweis auf dessen Alter, statt abzustürzen.

### Ansicht 1: Preisverlauf
- **F-4 (Muss):** Ein Diagramm zeigt den Stundenpreis (ct/kWh) für den gewählten Tag. Auswahl: heute, morgen (falls vorhanden), frei gewählter Tag.
- **F-5 (Muss):** Die günstigste und die teuerste Stunde sind im Diagramm markiert.
- **F-6 (Muss):** Der Nutzer stellt einen Schwellwert (ct/kWh) ein. Stunden darunter sind hervorgehoben.
- **F-7 (Soll):** Zusätzlich wird die Temperatur oder Sonnenscheindauer aus Open-Meteo im selben Diagramm oder daneben angezeigt.
- **F-8 (Kann):** Mehrtagesansicht (7 Tage) mit Tagesdurchschnitt.

### Ansicht 2: Statistiken
- **F-9 (Muss):** Für den gewählten Tag werden angezeigt: Durchschnittspreis, Minimum und Maximum (jeweils mit Uhrzeit), Standardabweichung und Anzahl Stunden unter dem Schwellwert.
- **F-10 (Soll):** Vergleich mit dem Vortag (Differenz des Durchschnittspreises in ct/kWh und in %).
- **F-11 (Kann):** Korrelation zwischen Preis und Sonnenscheindauer.

### Berechnung: Günstigstes Zeitfenster
- **F-12 (Muss):** Eingaben: Leistung `P` in kW, Dauer `T` in ganzen Stunden, Zeitraum (von/bis, innerhalb der verfügbaren Daten).
- **F-13 (Muss):** Das Programm findet den **zusammenhängenden** Block von `T` Stunden mit den niedrigsten Gesamtkosten.
  `Kosten = Σ (Stundenpreis in EUR/kWh) × P × 1 h`
- **F-14 (Muss):** Ausgegeben werden Startzeit, Endzeit, Kosten (EUR), Kosten bei Durchschnittspreis des Zeitraums und die Ersparnis (EUR).
- **F-15 (Soll):** Der gefundene Block ist im Diagramm farbig markiert.
- **F-16 (Kann):** Nicht zusammenhängende Variante: die `T` günstigsten Einzelstunden. Anzeige des Unterschieds zur zusammenhängenden Lösung.

## 3. Abnahmekriterien

Beispiel mit festen Stundenpreisen (EUR/MWh) von 00:00 bis 06:00: `80, 60, 40, 30, 35, 70`.
Eingabe: P = 11 kW, T = 3 h, Zeitraum 00:00–06:00.

| Messgrösse | Erwartung |
|---|---|
| Günstigstes Fenster | 02:00–05:00 (Preise 40, 30, 35) |
| Kosten im Fenster | 1.155 EUR |
| Kosten bei Durchschnittspreis (52.5 EUR/MWh) | 1.7325 EUR |
| Ersparnis | 0.5775 EUR |
| Durchschnitt, Minimum, Maximum der Preise | 52.5, 30 (03:00), 80 (00:00) EUR/MWh |
| Umrechnung 52.5 EUR/MWh | 5.25 ct/kWh |

Weitere Prüfungen:

| # | Prüfung | Erwartung |
|---|---|---|
| A-1 | Echte Daten für heute abrufen | Diagramm zeigt 24 Stundenwerte (23 oder 25 an Zeitumstellungstagen) |
| A-2 | 15-Minuten-Daten | werden zu Stundenmittelwerten aggregiert, Ergebnis identisch zur Rechnung per Hand |
| A-3 | Netzwerk getrennt oder API liefert 503 | App zeigt letzten Stand mit Alter, keine Fehlerseite |
| A-4 | T grösser als verfügbare Stunden | verständliche Fehlermeldung statt Absturz |

_Zu den Tabellen:_ Die Berechnung (Aggregation, Statistik, Fenstersuche) soll ohne API-Zugriff mit einer festen Preisreihe aufrufbar sein.

## 4. Abgrenzung (nicht gefordert)
Login, Datenbank, Echtzeit-Smart-Meter-Anbindung, Steuerung von Geräten, Deployment, CI.
