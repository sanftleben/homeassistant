# Training starten

Script, das den Denon AVR einschaltet, auf Spotify umschaltet und die Playlist "elektro" zufällig abspielt.

## Ablauf

1. Denon AVR einschalten und warten, bis er bereit ist (max. 30 s)
2. Eingang des AVR auf Spotify setzen
3. Spotify-Ausgabe auf das Spotify-Connect-Gerät des Denon legen
4. Shuffle aktivieren
5. Playlist abspielen

## Installation

Inhalt von [scripts/training_start.yaml](../scripts/training_start.yaml) einfügen über *Einstellungen → Automatisierungen & Szenen → Skripte → Skript hinzufügen → Neues Skript → ⋮ → In YAML bearbeiten*. Der Name wird beim Speichern vergeben.

Für `scripts.yaml` stattdessen den gesamten Inhalt unter einen Schlüssel wie `training_start:` stellen und um zwei Leerzeichen einrücken.

Im Block `variables` anpassen:

| Variable | Bedeutung |
|---|---|
| `avr` | Entity des Denon AVR |
| `spotify` | Entity der Spotify-Integration |
| `spotify_connect_device` | Name des Denon in Spotify Connect |
| `avr_source` | Name des Spotify-Eingangs am AVR |
| `playlist_uri` | `spotify:playlist:<id>` der Playlist "elektro" (Spotify → Playlist → Teilen → Link kopieren, ID steht zwischen `/playlist/` und `?`) |

## Alexa

1. Das Skript in Home Assistant für Alexa freigeben (Home Assistant Cloud oder Alexa-Integration → Entities freigeben). Es erscheint in Alexa als Szene.
2. In der Alexa-App unter *Routinen* eine neue Routine anlegen:
   - Wenn: Sprache → "Training beginnt"
   - Aktion: Smart Home → Szene → *Training starten*
3. Sprechen: "Alexa, Training beginnt"
