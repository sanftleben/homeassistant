# Notify Raumlüften

Home Assistant Blueprint für Lüftungshinweise anhand der Luftfeuchtigkeit.

## Funktionen

- Überwachung eines oder mehrerer Luftfeuchtigkeitssensoren
- Konfigurierbarer Luftfeuchtigkeits-Grenzwert
- Konfigurierbare Mindestdauer über dem Grenzwert, um kurze Messwertspitzen zu ignorieren
- Hysterese gegen Benachrichtigungs-Spam bei Werten rund um den Grenzwert
- Optionale Berücksichtigung von Außentemperatur und Außenluftfeuchte
- Lüftungsempfehlung nur, wenn die Außenluft tatsächlich trockener ist als die Raumluft
- Konfigurierbare Mindest-Außentemperatur als Frost- und Komfortgrenze
- Automatisches Nachholen der Empfehlung, sobald sich die Außenbedingungen bessern
- Berücksichtigung von Tür- und Fenstersensoren
- Keine Lüftungsempfehlung, wenn bereits ein Fenster oder eine Tür geöffnet ist
- Optionaler Hinweis, dass bereits gelüftet wird
- Optional erneute Benachrichtigung nach dem Schließen, wenn die Luftfeuchtigkeit weiterhin zu hoch ist
- Optional eine Entwarnung, sobald die Luftfeuchtigkeit wieder im Normalbereich liegt
- Beliebige Home-Assistant-`notify`-Entities als Ziel, z. B. Telegram, Alexa oder die Home Assistant Mobile App

## Installation

Die Blueprint-Datei kann direkt über Home Assistant importiert werden:

```text
https://raw.githubusercontent.com/sanftleben/homeassistant/main/automations/notify_raumlueften.yaml
```

Alternativ die Datei unter

```text
config/blueprints/automation/notify_raumlueften.yaml
```

ablegen und Home Assistant die Automationen bzw. Blueprints neu laden lassen.

## Konfiguration

### Luftfeuchtigkeitssensoren

Es können mehrere Sensoren ausgewählt werden. Für die Entscheidung, ob gelüftet werden soll, wird der höchste aktuell verfügbare Messwert verwendet.

### Grenzwert

Beispiel:

```text
Grenzwert: 60 %
```

Eine Benachrichtigung wird ausgelöst, wenn die Luftfeuchtigkeit den Wert für die konfigurierte Mindestdauer überschreitet.

### Mindestdauer

Standardmäßig 5 Minuten.

Damit wird verhindert, dass ein einzelner kurzer Messwertsprung sofort eine Benachrichtigung erzeugt.

### Hysterese

Standardmäßig 3 %.

Bei einem Grenzwert von 60 % und einer Hysterese von 3 % gilt:

- über 60 % → Lüften erforderlich
- unter 57 % → wieder normal
- zwischen 57 % und 60 % → Zustand bleibt unverändert

Dadurch wird verhindert, dass die Automation bei Messwerten wie `59 → 60 → 59 → 60 %` ständig zwischen den Zuständen wechselt.

### Außenbedingungen

Optional können ein Sensor für die Außentemperatur und ein Sensor für die Außenluftfeuchte ausgewählt werden. Sind beide gesetzt, wird eine Lüftungsempfehlung nur noch dann ausgesprochen, wenn Lüften physikalisch überhaupt etwas bringt.

Bleibt eines der beiden Felder leer oder liefert ein Sensor keinen gültigen Wert, verhält sich der Blueprint wie bisher und benachrichtigt ohne Prüfung der Außenbedingungen.

#### Warum nicht einfach die relative Luftfeuchtigkeit vergleichen?

Die relative Luftfeuchtigkeit sagt für sich genommen nichts darüber aus, wie viel Wasser die Luft tatsächlich enthält. Kalte Luft mit 95 % relativer Feuchte ist deutlich trockener als warme Luft mit 65 %.

Deshalb wird aus Temperatur und relativer Luftfeuchtigkeit die absolute Luftfeuchtigkeit in Gramm Wasser pro Kubikmeter berechnet — innen wie außen:

```text
innen   22 °C / 65 %  ->  12,6 g/m³
außen    8 °C / 95 %  ->   7,9 g/m³
```

Obwohl es draußen mit 95 % nach „sehr feucht" aussieht, wird beim Lüften Feuchtigkeit abtransportiert. Umgekehrt gilt:

```text
innen   22 °C / 65 %  ->  12,6 g/m³
außen   25 °C / 85 %  ->  19,6 g/m³
```

Hier würde Lüften die Feuchtigkeit im Raum sogar erhöhen. In diesem Fall wird keine Empfehlung gesendet.

#### Innentemperatur

Für die Berechnung wird die Raumtemperatur benötigt. Optional kann dafür ein Innentemperatursensor ausgewählt werden.

Ohne Sensor wird der Wert „Angenommene Innentemperatur" benutzt, standardmäßig 21 °C. Ein echter Sensor ist besonders in Räumen mit stark schwankender Temperatur sinnvoll, zum Beispiel im Bad nach dem Duschen.

#### Mindestunterschied absolute Luftfeuchtigkeit

Standardmäßig 1,0 g/m³.

Um so viel muss die Außenluft trockener sein als die Raumluft, damit gelüftet werden soll. Kleinere Werte machen die Empfehlung empfindlicher, größere zurückhaltender.

#### Mindest-Außentemperatur

Standardmäßig 0 °C.

Unterhalb dieser Temperatur wird nicht zum Lüften aufgefordert, auch wenn die Außenluft trockener wäre. Damit lassen sich Frost und unangenehme Auskühlung ausschließen.

#### Nachholen bei besserem Wetter

Ist die Luftfeuchtigkeit im Raum dauerhaft zu hoch und das Wetter zunächst ungeeignet, wird zunächst nichts gesendet.

Sobald die Außenbedingungen für die konfigurierte Mindestdauer passen und die Luftfeuchtigkeit weiterhin über dem Grenzwert liegt, wird die Empfehlung nachträglich ausgelöst.

### Tür- und Fenstersensoren

Alle ausgewählten Tür- und Fenstersensoren werden gemeinsam betrachtet.

Sobald mindestens einer davon geöffnet ist, gilt der Raum als bereits gelüftet. Eine normale Lüftungsempfehlung wird deshalb unterdrückt.

Wenn mehrere Fenster vorhanden sind, bleibt der Raum so lange im Lüftungszustand, wie mindestens eines davon geöffnet ist.

#### Beim Schließen geprüfte Bereiche

Optional können ein oder mehrere Home-Assistant-Bereiche ausgewählt werden. Wird ein Fenster oder eine Tür geschlossen, werden dann nur die Luftfeuchtigkeitssensoren und Tür-/Fenstersensoren aus diesen Bereichen geprüft. Das ist nützlich, wenn eine Automation Sensoren aus mehreren Räumen enthält und beim Schließen nicht jedes Mal alle Räume erneut geprüft werden sollen.

Bleibt das Feld leer, werden wie bisher alle Sensoren geprüft. Die übrigen Prüfungen, etwa die Lüftungsempfehlung oder die Entwarnung, verwenden immer alle Sensoren.

### Benachrichtigungen

Als Ziel werden Home-Assistant-`notify`-Entities ausgewählt. Dadurch ist der Blueprint nicht auf einen bestimmten Messenger festgelegt.

Beispiele:

- Telegram
- Home Assistant Mobile App
- Alexa
- andere `notify`-Integrationen

Mehrere Ziele können gleichzeitig ausgewählt werden.

## Verhalten

```text
                 Luftfeuchtigkeit > Grenzwert
                              │
                     Außenluft trockener?
                     (und nicht zu kalt)
                         /            \
                       nein            ja
                        │              │
                        │      Mindestdauer erreicht?
                        │          /          \
                        │        nein          ja
                        │         │             │
                        │         │      Fenster/Tür offen?
                        │         │        /          \
                        │         │      ja            nein
                        │         │      │               │
                        │         │      │        Benachrichtigung
                        │         │      │               │
                        │         │      └── Lüften ─────┘
                        │         │
                        └─────────┴──────── warten

Fenster/Tür wird geschlossen
              │
              ▼
       noch über Grenzwert?
          /           \
        ja             nein
        │                │
        ▼                ▼
  Außenluft noch      nichts
   geeignet?
   /        \
  ja         nein
  │            │
  ▼            ▼
erneut       nichts
melden
```

Die Entwarnung verwendet die Hysterese. Bei 60 % Grenzwert und 3 % Hysterese wird erst unter 57 % eine Normalmeldung ausgelöst.

Die Prüfung der Außenbedingungen gilt nur für die eigentlichen Lüftungsempfehlungen. Die Entwarnung und der Hinweis „es wird bereits gelüftet" sind Statusmeldungen und werden davon nicht unterdrückt.

## Beispiel

Konfiguration:

- Schlafzimmer
- Luftfeuchtigkeit: 60 %
- Mindestdauer: 5 Minuten
- Hysterese: 3 %
- Innentemperatur: Sensor Schlafzimmer
- Außentemperatur: Sensor Garten
- Außenluftfeuchte: Sensor Garten
- Mindestunterschied: 1,0 g/m³
- Mindest-Außentemperatur: 0 °C
- Fensterkontakt: Schlafzimmerfenster
- Benachrichtigung: Telegram

Ablauf:

1. Luftfeuchtigkeit steigt auf 61 % bei 22 °C Raumtemperatur.
2. Draußen sind es 25 °C bei 85 %. Die Außenluft ist feuchter als die Raumluft, es wird nichts gesendet.
3. Am Abend kühlt es auf 12 °C bei 80 % ab. Die Außenluft ist jetzt deutlich trockener.
4. Nach fünf Minuten unter diesen Bedingungen wird eine Telegram-Nachricht gesendet.
5. Das Fenster wird geöffnet.
6. Es wird keine weitere normale Lüftungsaufforderung gesendet.
7. Das Fenster wird geschlossen.
8. Liegt die Luftfeuchtigkeit noch über 60 % und ist die Außenluft weiterhin geeignet, wird erneut zum Lüften aufgefordert.
9. Sinkt sie anschließend unter 57 %, ist der Zustand wieder normal.

## Hinweis zu mehreren Sensoren

Wenn mehrere Luftfeuchtigkeitssensoren ausgewählt sind, wird für die Benachrichtigung und die Lüftungsentscheidung der höchste gültige Messwert verwendet. So löst beispielsweise ein feuchter Sensor im Badezimmer auch dann eine Meldung aus, wenn ein zweiter Sensor einen niedrigeren Wert meldet.
