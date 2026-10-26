# Dunstabzug – Koch-Erkennung per Temperaturanstieg

Home Assistant Blueprint zur automatischen Steuerung eines Dunstabzugs. Die Koch-Erkennung ist für einen Gasherd ausgelegt, bei dem eine elektrische Leistungsaufnahme kaum als Signal verwendbar ist. Als Signal dient die Abwärme, gemessen an einem Klimasensor in der Nähe des Dunstabzugs.

## Erkennung

Die Automation wertet ausschließlich die **Änderungsrate der Temperatur** aus. Absolute Werte werden bewusst nicht verwendet, weil sie vom normalen Küchenklima abhängen. Die Luftfeuchtigkeit wird nicht ausgewertet: Am Sensor direkt am Dunstabzug sinkt sie beim Kochen eher, weil sich die Luft erwärmt.

| Signal | Standard |
|---|---:|
| Einschalten ab Anstieg von | 0,5 °C/min |
| Ausschalten, wenn die Rate unter (Abstieg) | −0,25 °C/min |

Die Standardwerte stammen aus dem Temperaturverlauf vom 03.–10.10.2026:

| Typ | Dauer | maximale Rate (3-Minuten-Fenster) |
|---|---:|---:|
| Kochen (6 Vorgänge) | 8–15 min | 0,74 – 2,6 °C/min |
| Kurze Störungen (3 Fälle) | 3–4 min | 0,5 – 0,6 °C/min |

Die Gruppen überlappen sich. Der schwächste Kochvorgang (10.10. abends, +4 °C in 8 Minuten) erreichte nur 0,74 °C/min und wurde mit der früheren Schwelle 0,8 verpasst. Mit 0,5 werden alle sechs Kochvorgänge erkannt, dafür kann auch eine der Störungen (Anstieg über etwa 3 Minuten) den Dunstabzug kurz einschalten. Er geht dann wieder aus, sobald die Temperatur fällt. Bei verpassten Kochvorgängen die Schwelle senken, bei Fehlalarmen anheben oder die Bestätigungsdauer erhöhen.

## Schutz gegen Fehlauslösung beim Betreten

Das blosse Betreten der Küche kann den Dunstabzug nicht starten. Anwesenheit ist reine Bedingung und niemals Auslöser für den Anstieg. Zwei Zeitschranken müssen zusätzlich erfüllt sein:

| Zeitschranke | Standard | Bedeutung |
|---|---:|---|
| Anwesenheits-Vorlaufzeit | 3 min | So lange muss die Küche bereits durchgehend belegt sein, bevor eingeschaltet werden darf |
| Bestätigungsdauer | 1 min | So lange muss die Änderungsrate ununterbrochen über der Koch-Schwelle liegen |

Hat das Kochen schon vor der Anwesenheitserkennung begonnen, wird nach Ablauf der Vorlaufzeit erneut geprüft, ob die Rate über der Schwelle liegt.

Falls Fehlalarme auftreten, zuerst die Bestätigungsdauer erhöhen, danach die Schwelle anheben.

## Nachlauf

Der Dunstabzug schaltet aus, wenn die Temperatur **fällt**: Die Änderungsrate muss die Nachlaufzeit lang durchgehend unter der negativen Abschalt-Schwelle liegen. Standard sind **2 Minuten** bei **−0,25 °C/min**.

Beim Kochen bleibt die Temperatur am Sensor oft auf einem Plateau (Rate nahe 0). Ein Plateau schaltet deshalb nicht aus, erst der Abfall nach dem Kochen. In den Messdaten liegt die Rate auf dem Plateau zwischen 0 und ±0,18 °C/min (Quantisierung des Sensors in 0,5-°C-Schritten). Nach dem Kochen fällt sie mehrere Minuten auf −0,34 bis −1,5 °C/min. Die Schwelle −0,25 liegt in dieser Lücke.

Kehrt die Rate während des Nachlaufs über die Schwelle zurück, beginnt der Nachlauf neu. Schaltet der Dunstabzug bei einem zeitweise sinkenden Kochen (z. B. Flamme klein gedreht) zu früh aus, die Nachlaufzeit erhöhen oder die Schwelle senken. Kühlt die Küche nach dem Kochen nur langsam ab, schaltet erst die späteste Einschaltzeit aus.

Zusätzlich prüft die späteste Einschaltzeit einmal täglich, ob abgeschaltet werden kann. Das dient als Sicherheitsnetz, falls die Automation neu geladen wurde, während der Dunstabzug lief.

## Fenster

- **Fenster öffnen:** Läuft der Dunstabzug, schaltet er sofort aus. Bei offenem Fenster schaltet er auch bei Kochanstieg nicht ein.
- **Fenster schliessen:** Der Dunstabzug schaltet wieder ein, wenn alle Punkte zutreffen:
  - Er wurde durch das Öffnen ausgeschaltet. Das wird daran erkannt, dass sich sein Zustand nach dem Öffnen geändert hat. War er vorher schon aus (Nachlauf oder von Hand), bleibt er aus.
  - Die Küche ist belegt.
  - Das Fenster war höchstens 20 Minuten offen (einstellbar).
  - Es liegt zwischen frühester und spätester Einschaltzeit.

„Es wird noch gekocht“ wird dabei **nicht gemessen**, sondern aus Anwesenheit und kurzer Öffnungsdauer geschlossen. Bei offenem Fenster fällt die Temperatur, daher sagt die Änderungsrate in dieser Zeit nichts über das Kochen aus. Wurde in der Zwischenzeit zu Ende gekocht, läuft der Dunstabzug nach dem Schliessen an, bis die Temperatur wieder fällt. Im ungünstigsten Fall schaltet ihn erst die späteste Einschaltzeit aus.

## Benötigte Sensoren

1. Ein Temperatursensor der Küche, bevorzugt in der Nähe des Dunstabzugs
2. Ein **Derivative-Sensor** (Helfer „Ableitung“) auf diesem Sensor, Einheit pro Minute
3. Ein Anwesenheitssensor der Küche
4. Ein Fensterkontakt der Küche (`on` = offen)
5. Der zu steuernde Switch bzw. Fan

Der Blueprint erwartet im Feld *Temperatur – Änderungsrate* den Derivative-Sensor. Wird dort versehentlich der Rohsensor (z. B. 22 °C) eingetragen, liegt der Wert immer über der Schwelle: Die Rate-Trigger feuern nie, und der Dunstabzug geht nie automatisch aus.

### Derivative-Sensor

Als Helfer unter **Einstellungen → Geräte & Dienste → Helfer → Helfer hinzufügen → Ableitungssensor**:

| Feld | Wert |
|---|---|
| Eingangssensor | Temperatursensor der Küche |
| Zeitfenster | 3 Minuten |
| Zeiteinheit | Minuten |
| Genauigkeit | 2 |

Oder in YAML:

```yaml
sensor:
  - platform: derivative
    source: sensor.kuche_temperatur
    name: Küche Temperatur Änderungsrate
    unit_time: min
    time_window: '00:03:00'
```

Das 3-Minuten-Fenster glättet kurze Messspitzen. Die Schwellen im Blueprint wurden mit diesem Fenster ermittelt. Bei einem anderen Fenster ändern sich die Raten.

## Blueprint installieren

Raw-URL:

`https://raw.githubusercontent.com/sanftleben/homeassistant/main/automations/dunstabzug_automatisierung.yaml`

Danach in Home Assistant unter **Einstellungen → Automatisierungen & Szenen → Blueprints** eine neue Automation aus dem Blueprint erstellen.

## Verifikation im Betrieb

Die Automation lässt sich nur im laufenden Home Assistant sinnvoll prüfen. Unter **Einstellungen → Automatisierungen → Dunstabzug → Traces**:

1. **Betreten ohne Kochen** darf keinen Trace im Zweig *Dunstabzug einschalten* erzeugen.
2. **Kochbeginn** erzeugt einen Trace mit dem Zweig *Dunstabzug einschalten*.
3. **Kochende** erzeugt nach Ablauf der Nachlaufzeit einen Trace mit dem Zweig *Dunstabzug ausschalten*.

Ein abgebrochener Trace im Einschaltzweig ist normal: er bedeutet, dass ein Anstieg zwar lange genug anhielt, aber Zeitfenster, Anwesenheit oder Schwelle nicht erfüllt waren.
