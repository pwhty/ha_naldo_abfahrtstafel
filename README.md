# ha-naldo-abfahrtstafel

Home-Assistant-Konfiguration zur Darstellung einer **NALDO-Abfahrts- und Ankunftstafel** (ÖPNV) auf Basis der öffentlichen EFA-Schnittstelle von naldo.de.

Die Lösung besteht aus:
- REST-Sensoren zum Abruf der Rohdaten
- Template-Sensoren zur Aufbereitung
- Jinja-Macros zur Datenformatierung
- einer Lovelace-Card für die visuelle Anzeige

<img width="525" height="410" alt="image" src="https://github.com/user-attachments/assets/b40c938a-751b-4abe-b4b1-3da5a984ac78" />
<img width="525" height="410" alt="image" src="https://github.com/user-attachments/assets/1228260d-9e24-429d-b721-8a1eb8c8f166" />

---

## ✨ Features

- Anzeige der **nächsten Abfahrten und Ankünfte** je Haltestelle
- **Realtime-Daten inkl. Verspätungen**
- Trennung von **Rohdaten** und **UI-Logik**
- Kompatibel mit  
  `custom:public-transport-departures-card`
- Mehrere Haltestellen einfach erweiterbar

---

## 🧩 Architektur-Überblick

NALDO EFA API

↓

REST Sensor (Rohdaten)

↓

Template Sensor

↓

Jinja Formatter (Macros)

↓

Lovelace UI (Abfahrtstafel)

---

## 🛠 Voraussetzungen

- Home Assistant (Core / OS / Supervised)
- Aktivierte **REST-Integration**
- Zugriff auf lokale Jinja-Templates (`/config/templates`)
- Folgende Custom Cards:
  - `custom:public-transport-departures-card`
  - `custom:tabbed-card`
  - `custom:bubble-card`

---

## 📡 NALDO API / Haltestellen-ID

Für den Zugriff ist **kein API-Key notwendig**.

Die Haltestelle wird über die sogenannte **Infra-ID** im GET-Parameter `name_dm` angegeben.

### Quelle der IDs
Offizielle CSV: https://www.nvbw.de/fileadmin/user_upload/service/open_data/haltestellen/haltestellen.csv

### Beispiel
**Pfullingen – Laiblinsplatz**
ID: de:08415:29010
URL-kodiert im Request: name_dm=de%3A08415%3A29010

---

## 🔧 Wichtige GET-Parameter

| Parameter | Bedeutung |
|---------|----------|
| `itdDateTimeDepArr=dep` | Abfahrten |
| `itdDateTimeDepArr=arr` | Ankünfte |
| `depSequence=10` | Anzahl der Einträge |
| `useRealtime=1` | Echtzeitdaten |
| `outputFormat=rapidJSON` | JSON-Antwort |

---

## ⚙️ REST-Sensoren (Rohdaten)

Beispiel: **Abfahrten Ahlsberg**

```yaml
rest:
  - resource: https://www.naldo.de/...&itdDateTimeDepArr=dep&name_dm=de%3A08415%3A29000
    scan_interval: 60
    sensor:
      - name: "Naldo Rohdaten Abfahrt Ahlsberg"
        unique_id: naldo_raw_departures_ahlsberg
        value_template: "OK"
        json_attributes:
          - stopEvents
```

Der Sensor speichert alle relevanten Daten in stopEvents.

---

## 🧠 Template-Sensoren (aufbereitete Daten)

```yaml
template:
  - sensor:
      - name: "Naldo Abfahrt Ahlsberg"
        unique_id: naldo_abfahrt_ahlsberg
        state: >
          {% from 'naldo/naldo_formatter.jinja' import format_naldo_departure_response_for_state %}
          {{ format_naldo_departure_response_for_state(
            state_attr('sensor.naldo_rohdaten_abfahrt_ahlsberg', 'stopEvents')
          ) }}
        unit_of_measurement: "min"
        attributes:
          departures: >
            {% from 'naldo/naldo_formatter.jinja' import format_naldo_departure_response_for_departures_ui %}
            {{ format_naldo_departure_response_for_departures_ui(
              state_attr('sensor.naldo_rohdaten_abfahrt_ahlsberg', 'stopEvents')
            ) }}
```

-	State → Minuten bis zur nächsten Abfahrt
-	Attribut departures → Liste für die UI-Karte

---

## 🧩 Jinja-Macros

/config/templates/naldo/naldo_formatter.jinja

Enthaltene Macros:
- format_naldo_departure_response_for_state
-	format_naldo_arrival_response_for_state
-	format_naldo_departure_response_for_departures_ui
-	format_naldo_arrival_response_for_departures_ui

Die Macros:
-	berechnen Verspätungen
-	normalisieren Verkehrs­mittel
-	liefern strukturierte JSON-Objekte für Lovelace

---

## 🖥 Lovelace UI

Beispiel-Dashboard (card.yaml):
-	Tabs für Abfahrt / Ankunft
-	Farbliche Hervorhebung je nach Zeit bis Ereignis
-	Detailtabelle mit Ziel, Linie, Verspätung und Zwischenhalten

```yaml
type: custom:public-transport-departures-card
entity: sensor.naldo_abfahrt_ahlsberg
departures_attribute: departures
departure_properties:
  time: zeit
  delay: verspaetung
  train: verkehrsmittel
  direction: ziel
  next_stations: ueber
```

---

➕ Weitere Haltestellen hinzufügen
1.	Neue REST-Resource mit anderer name_dm
2.	Eigenen Rohdaten-Sensor anlegen
3.	Template-Sensor referenziert neuen Rohdaten-Sensor
4.	Karte duplizieren

---

⚠️ Hinweise
-	Die API ist inoffiziell, Änderungen sind möglich
-	scan_interval nicht unnötig niedrig setzen
-	Templates werden nur nach Neustart geladen

---

📄 Lizenz

MIT License

---

🙌 Credits
-	Datenquelle: NALDO / EFA
-	UI-Card: public-transport-departures-card
-	Idee & Umsetzung: pwhty

---
