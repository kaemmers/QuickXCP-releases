# QuickXCP – Handbuch

QuickXCP ist ein schlankes Messtool für Steuergeräte über **XCP on CAN**. Variablen kommen
direkt aus dem **ELF** (DWARF), die Verbindungsdaten aus der **A2L**. Es gibt keine Messgrößen
anzulegen und keine A2L zu pflegen: Variable suchen, auf die Messseite ziehen, **F9** drücken.

**Download:** [neueste Version](https://github.com/kaemmers/QuickXCP-releases/releases/latest)
(`QuickXCP-Setup-<Version>.exe`) · Dieses Handbuch liegt auch im Programmordner
(`HANDBUCH.md`).

## Inhalt

1. [Was QuickXCP kann](#1-was-quickxcp-kann)
2. [Installieren und aktualisieren](#2-installieren-und-aktualisieren)
3. [Schnellstart](#3-schnellstart)
4. [Konfiguration: Geräteprofil anlegen](#4-konfiguration-geräteprofil-anlegen)
5. [Projekt anlegen](#5-projekt-anlegen)
6. [Messen](#6-messen)
7. [Messseiten](#7-messseiten)
8. [Plot, Cursor und Marker](#8-plot-cursor-und-marker)
9. [Aufzeichnen und Aufzeichnungen ansehen](#9-aufzeichnen-und-aufzeichnungen-ansehen)
10. [Parameter schreiben](#10-parameter-schreiben)
11. [Tastenkürzel und Maus](#11-tastenkürzel-und-maus)
12. [VS-Code-Erweiterung](#12-vs-code-erweiterung)
13. [Claude-Skill: Messseite aus Testplan](#13-claude-skill-messseite-aus-testplan)
14. [Kommandozeile](#14-kommandozeile-quickxcp-cliexe)
15. [Dateien und Ablageorte](#15-dateien-und-ablageorte)
16. [Fehlersuche](#16-fehlersuche)
17. [Änderungshistorie](#17-änderungshistorie)

---

## 1. Was QuickXCP kann

| Bereich | Möglichkeiten |
|---|---|
| **Variablen** | direkt aus dem ELF (DWARF 2 und neuer): globale und statische Variablen, Struct-Member, Arrays, Bitfelder, Enums. Volltextsuche mit mehreren Wörtern. Großes ELF wird beim ersten Mal parallel gelesen (Sekunden), danach aus dem Cache. |
| **Verbindung** | XCP on CAN über Vector, PEAK (PCAN), Kvaser oder IXXAT. CAN-IDs, Bitrate, Timeouts, Events und DAQ-Daten aus der A2L (`IF_DATA XCPplus`); Bitrate überschreibbar. Simulator zum Ausprobieren ohne Hardware. |
| **Messen** | ein Knopf: **F9** verbindet und misst, **Esc** stoppt. Polling (Rate je Variable) und **Dynamic DAQ** (Events aus der A2L, bis 1 ms). Automatisches Wiederverbinden nach Steuergeräte-Neustart. |
| **Anzeige** | Messtabelle mit erreichter Rate und Mini-Kurven, alternativ große **Anzeige-Kacheln**; Dezimal/Hex/Binär; physikalische Werte, Einheiten und Texttabellen aus der A2L. |
| **Plot** | bis zu 4 Plotbereiche mit gemeinsamer Zeitachse, Zoom, zwei Cursor mit Δt/Δy, Marker, ganze Messung im Speicher. |
| **Aufzeichnung** | MDF4 (`.mf4`, in CANape/asammdf lesbar) oder CSV; eigene und fremde Aufzeichnungen wieder anzeigen. |
| **Schreiben** | Parameter im RAM schreiben – standardmäßig gesperrt, mit Prüfung gegen die Speicherkarte, Rücklesen und Schreib-Log. Flash und Peripherie werden nie beschrieben. |
| **Organisation** | **Projekte** mit mehreren **Messseiten** (farbige Reiter), eine Datei pro Seite, Git-tauglich; Seiten im- und exportierbar. |
| **Sicherheit** | ELF/Firmware-Abgleich per Prüfsumme, Warnung bei anderem XCP-Master auf dem Bus, Warnung ab 60 % Buslast. |
| **Werkzeuge** | **VS-Code-Erweiterung** (messen, Live-Werte und Plots direkt im C-Code), **Claude-Skill** (Messseite aus Testplan), Kommandozeile. |
| **Bedienung** | helles/dunkles Design, frei andockbare Bereiche mit Layout-Vorlagen, Befehlssuche (Strg+P), Rückgängig für die Messseite, automatische Updates. |

---

## 2. Installieren und aktualisieren

### Voraussetzungen

- **Windows 10 oder 11, 64 Bit (x64).** Windows auf ARM wird nicht unterstützt (die
  CAN-Treiber der Hersteller fehlen dort meist).
- **CAN-Interface** von Vector, PEAK, Kvaser oder IXXAT mit **aktuellem Herstellertreiber**.
  Unter Windows 11 blockiert die *Speicher-Integrität* (Kernisolierung) ältere Treiber –
  das Interface taucht dann nicht auf.
- **Vector:** zusätzlich die XL-Library `vxlapi64.dll`. Sie kommt mit dem Vector-Treiber
  bzw. dem *Vector XL Driver Library*-Paket. Sie wird aus Lizenzgründen nicht mitgeliefert;
  alternativ in den Unterordner `vector\` neben `QuickXCP.exe` legen.
- Keine Python-Installation, keine Administratorrechte.

### Installieren

1. `QuickXCP-Setup-<Version>.exe` von der
   [Release-Seite](https://github.com/kaemmers/QuickXCP-releases/releases/latest) laden
   und starten.
2. Die exe ist **nicht signiert**: SmartScreen meldet „Der Computer wurde durch Windows
   geschützt“ → *Weitere Informationen* → *Trotzdem ausführen*. Ist unter Windows 11
   *Intelligente App-Steuerung* aktiv, blockiert Windows das Programm vollständig.
3. Installiert wird pro Benutzer nach `%LOCALAPPDATA%\Programs\QuickXCP`. Optionen:
   Desktop-Symbol, **Erweiterung für Visual Studio Code** (nur sichtbar, wenn VS Code
   installiert ist; vorgewählt).

### Updates

QuickXCP prüft beim Start, ob es eine neue Version gibt (abschaltbar unter
*Hilfe → Beim Start nach Updates suchen*; von Hand: *Hilfe → Nach Updates suchen …*).
Ein Update lädt den Installer, prüft dessen SHA-256-Prüfsumme, installiert still und
startet QuickXCP neu. Profile, Projekte, Einstellungen und der Ordner `vector\` bleiben
erhalten; die VS-Code-Erweiterung wird mit aktualisiert.

### Deinstallieren

*Windows-Einstellungen → Apps* → *QuickXCP*. Persönliche Daten unter
`%APPDATA%\QuickXCP` bleiben stehen und können von Hand gelöscht werden.

---

## 3. Schnellstart

**Ohne Steuergerät (Simulator):**

1. QuickXCP starten, *Datei → Demo-ELF erzeugen und laden*.
2. In der Werkzeugleiste *Schnittstelle = Simulator*.
3. **Strg+P**, „motor“ tippen, Variable mit Enter übernehmen (oder Variablenbaum mit
   **Strg+B** öffnen und per Doppelklick hinzufügen).
4. **F9** – Werte laufen, **1** legt die markierte Variable in Plotbereich 1.

**Mit Steuergerät:**

1. **ELF** öffnen (*Datei → ELF öffnen*, Strg+O, oder Datei ins Fenster ziehen) – das ELF
   genau der Software, die im Steuergerät läuft.
2. **A2L** öffnen (*Datei → A2L (Verbindungsdaten) öffnen*). Sie liefert CAN-IDs, Bitrate
   und Events. Ohne A2L gelten die Werte aus dem Geräteprofil.
3. **Schnittstelle** und **Kanal** in der Werkzeugleiste wählen (oder einmal im
   [Geräteprofil](#4-konfiguration-geräteprofil-anlegen) hinterlegen). Bitrate „aus A2L“
   lassen, außer der Bus läuft anders.
4. Variablen hinzufügen, **F9**. Die Statusleiste zeigt Verbindung, Buslast, Bitrate,
   CAN-IDs, Events und das aktive Profil.
5. CANape oder andere XCP-Master vorher beenden – es darf nur einen Master geben.

---

## 4. Konfiguration: Geräteprofil anlegen

Alles, was zum **Arbeitsplatz** und zum **Steuergerätetyp** gehört, steht in einem
**Geräteprofil** (TOML-Datei): CAN-Interface und Kanal, die **Speicherkarte** für den
Schreibschutz und Ersatzwerte für CAN-IDs und Events, solange keine A2L geladen ist.
**Die A2L hat bei den XCP-Daten immer Vorrang.**

Ohne eigenes Profil gilt ein neutrales Standardprofil. Für das **Schreiben** von
Parametern ist ein eigenes Profil mit korrekter Speicherkarte dringend empfohlen.

### Anlegen

1. Vorlage kopieren: `%LOCALAPPDATA%\Programs\QuickXCP\profiles_example\beispiel.toml`
2. ablegen unter `%APPDATA%\QuickXCP\profiles\<name>.toml` (Ordner ggf. anlegen) –
   oder in `profiles\` neben `QuickXCP.exe`. Dort wird es automatisch gefunden.
3. anpassen (siehe unten), QuickXCP neu starten oder *Datei → Geräteprofil laden …*.
   Das aktive Profil steht rechts in der Statusleiste.

### Aufbau

```toml
name = "Mein Steuergerät"

[can]
interface = "vector"     # vector | pcan | kvaser | ixxat | socketcan
vn_channel = 1           # Vector: Kanal am Gerät, wie beschriftet (1..n)
# vn_serial = 123456     # nur bei mehreren Vector-Geräten
# channel = "0"          # Vector: Kanal der App "QuickXCP" im Hardware Manager (0 = CAN 1)
#                        # andere: z. B. "PCAN_USBBUS1"
bitrate = 500000         # nur ohne A2L

[xcp]                    # nur ohne A2L
master_id = 0x7F0
slave_id = 0x7F1
extended = false
events = [
  { name = "10ms",  channel = 0, cycle_ms = 10 },
  { name = "100ms", channel = 1, cycle_ms = 100 },
]

# Speicherkarte: "ram" = schreibbar, "flash" / "sfr" = nie schreiben.
# Adressen außerhalb aller Bereiche werden ebenfalls nie beschrieben.
[[memory]]
name = "RAM"
start = 0x70000000
size = 0x3C000
kind = "ram"

[[memory]]
name = "Program Flash"
start = 0x80000000
size = 0x10000000
kind = "flash"
```

| Abschnitt | Schlüssel | Bedeutung |
|---|---|---|
| — | `name` | Anzeigename in der Statusleiste |
| `[can]` | `interface` | Treiber-Backend |
| | `vn_channel`, `vn_serial` | Vector: Kanal direkt am Gerät (empfohlen) |
| | `channel` | Vector-App-Kanal bzw. Kanalname anderer Interfaces |
| | `bitrate` | Bit/s, nur ohne A2L |
| `[xcp]` | `master_id`, `slave_id`, `extended` | CAN-IDs, nur ohne A2L |
| | `events` | DAQ-Events, nur ohne A2L |
| `[[memory]]` | `name`, `start`, `size`, `kind` | Speicherbereiche aus Map-Datei/Linker-Skript |

Die Speicherkarte nutzt QuickXCP auch für den ELF/Firmware-Abgleich (Flash-Bereiche).

**Persönliche Einstellungen** (zuletzt gewählte Schnittstelle, Kanal, Bitrate, Design,
Aufzeichnungsformat, zuletzt geöffnetes Projekt …) merkt sich QuickXCP automatisch in
`%APPDATA%\QuickXCP\session.json` – dafür ist kein Profil nötig.

---

## 5. Projekt anlegen

Ein **Projekt** fasst ELF, A2L und alle **Messseiten** zusammen. Ohne eigenes Projekt
arbeitet QuickXCP im Standardprojekt unter `%APPDATA%\QuickXCP\Projekt\`.

1. ELF und A2L öffnen (werden ins neue Projekt übernommen).
2. *Datei → Neues Projekt …*, Ordner und Namen wählen, z. B. `MeinProjekt\projekt.qxp`.
   Das Projekt startet mit „Messseite 1“.
3. Messseiten anlegen und füllen – alles wird automatisch gespeichert.

Öffnen: *Datei → Projekt öffnen …*, über *Datei → Zuletzt verwendet* oder die `.qxp` ins
Fenster ziehen. *Projekt speichern unter …* kopiert Projekt und Seiten an einen neuen Ort.

```
MeinProjekt/
  projekt.qxp          ELF- und A2L-Pfad (relativ zum Projektordner)
  seiten/
    motor.qxpage       eine Datei pro Messseite
    diagnose.qxpage
  Messungen/           Aufzeichnungen (Standard-Ablageordner)
```

**Im Team / mit Git:** Liegen ELF und A2L im Projekt oder daneben, funktionieren die
relativen Pfade auf jedem Rechner. Jede Seite ist eine eigene Datei – legen zwei
Kollegen gleichzeitig Seiten an, gibt es keinen Merge-Konflikt. Dateien werden nur
geschrieben, wenn sich ihr Inhalt ändert. Persönliches (Schnittstelle, Design …) steht
nicht im Projekt.

---

## 6. Messen

### Verbindung

| Werkzeugleiste | |
|---|---|
| **Schnittstelle** | Simulator, Vector, PCAN, Kvaser, IXXAT … |
| **Kanal** | bei Vector „VN… Channel N (direkt)“ wie am Gerät beschriftet, oder der Kanal einer im *Vector Hardware Manager* angelegten Anwendung **QuickXCP** |
| **Bitrate** | „aus A2L“ oder fest (überschreibt die A2L) |
| **Messung starten** | F9 – verbinden, Polling sofort, DAQ für alle Variablen mit Event |

**Esc** stoppt und trennt (eine laufende Aufzeichnung wird geschlossen). Startet das
Steuergerät neu, verbindet QuickXCP automatisch wieder, sobald es antwortet. Der Zustand
ist auch in der Taskleiste zu sehen (grün = misst, orange = Verbindung verloren).

### Variablen hinzufügen

- **Strg+P** (Befehls- und Variablensuche) oder **Strg+F** (Variablensuche): mehrere
  Wörter = alle müssen vorkommen, z. B. `motor speed`.
- **Variablenbaum** (Strg+B): Doppelklick = in die Tabelle; Ziehen auf einen Plotbereich
  = in Tabelle und Plot. Ein Struct oder Array bringt alle Elemente mit (ab 64 Elementen
  fragt QuickXCP nach).
- Aus **VS Code**: Rechtsklick auf die Variable → *In QuickXCP messen* (siehe
  [Abschnitt 12](#12-vs-code-erweiterung)).
- Pfadschreibweise: `g_motor.speed`, `g_wheels[2].pressure`, funktionsstatische Variablen
  `Funktion::name`, gleichnamige dateistatische `name@datei`.

### Erfassung: Polling oder DAQ

In der Spalte **Erfassung** je Variable:

- **Polling** – QuickXCP fragt mit der eingestellten *Rate [ms]* ab. Gut für Parameter
  und langsame Größen.
- **Event (DAQ)** – das Steuergerät sendet die Werte selbst im Takt des Events (aus der
  A2L). Für schnelle Signale und Flanken, die man nicht verpassen darf. *Rate* zeigt dann
  ausgegraut den Event-Takt.

*erreicht [Hz]* zeigt die tatsächliche Rate. Änderungen während der Messung werden sofort
übernommen (DAQ wird neu aufgesetzt). Mehrere Zeilen markieren und eine davon ändern =
gilt für alle markierten. 64-Bit-Werte sind mit ⚠ markiert: Sie werden in zwei Teilen
gelesen und sind nicht garantiert konsistent.

### Tabelle und Anzeige

- **Messtabelle**: Wert, Typ, Adresse, Rate, Mini-Kurve (Strg+K). Neue/geänderte Werte
  stehen kurz **fett**. Strg+Mausrad = Schriftgröße, Spalten per Ziehen verschieben und
  per Rechtsklick auf den Kopf ein-/ausblenden. Zeilen per Ziehen oder Alt+↑/↓ umsortieren.
- **Große Anzeige** (Strg+Umschalt+A): Kacheln mit großen Werten, z. B. für den
  Prüfstand; Vollbild mit Umschalt+F11.
- **Rechtsklick** auf Zeilen/Kacheln: Erfassung, Plotbereich, Anzeige Dez/Hex/Bin,
  physikalisch/roh (A2L-Umrechnung), Entfernen.

### Anordnung

Variablen, Tabelle, Plot und Log sind Bereiche, die sich frei andocken lassen (Titel
ziehen), auch als eigenes Fenster. F11 maximiert den aktiven Bereich, Rechtsklick auf
einen Titel bietet Layout-Vorlagen. Die Anordnung wird je Messseite gemerkt.

### Sicherheitshinweise beim Messen

- **ELF-Abgleich**: Nach dem Verbinden vergleicht QuickXCP Prüfsummen der Flash-Bereiche
  des ELF mit dem Steuergerät und warnt, wenn das ELF nicht zur Software passt – dann
  stimmen Adressen nicht (*Werkzeuge → ELF-Abgleich beim Verbinden*).
- **Neues ELF**: Wird das ELF neu gebaut, lädt QuickXCP es nach und löst alle Variablen
  neu auf. Variablen, die es nicht mehr gibt, bleiben ausgegraut mit ihren Einstellungen
  stehen.
- **Anderer XCP-Master** auf denselben CAN-IDs (z. B. CANape läuft noch): Warnung in
  Statuszeile und Log.
- **Buslast**: orange ab 60 %, rot ab 80 %.

---

## 7. Messseiten

Eine **Messseite** ist eine Watch-Liste mit ihren Plots und ihrer Anordnung. Mehrere Seiten
erscheinen als farbige Reiter; gemessen wird die aktive Seite. Die Seite lässt sich
während der Messung wechseln (Polling und DAQ ziehen mit), nur während einer
Aufzeichnung nicht.

| Aktion | |
|---|---|
| neue Seite | **+** am Reiter oder Strg+T |
| wechseln | Klick, Strg+Tab / Strg+Umschalt+Tab |
| umbenennen | Doppelklick auf den Reiter |
| Reihenfolge | Reiter ziehen |
| Farbe, Duplizieren, Exportieren, Importieren, Löschen | Rechtsklick auf den Reiter |
| Seite eines Kollegen übernehmen | `.qxpage`-Datei ins Fenster ziehen |
| Rückgängig / Wiederholen | Strg+Z / Strg+Y |

Messseiten speichern nur **Namen** von Variablen, keine Adressen – sie funktionieren
deshalb mit jedem neuen Build weiter. Das Dateiformat ist einfaches TOML und lässt sich
auch von Hand oder vom [Claude-Skill](#13-claude-skill-messseite-aus-testplan) erzeugen:

```toml
format = 1
name = "Drehzahl"

[[signal]]
path = "g_motor.speed"
event = 1          # DAQ-Event-Kanal, -1 = Polling
period_ms = 100    # Polling-Rate
plot = true
area = 0           # Plotbereich 0..3
fmt = "dec"        # dec | hex | bin
```

---

## 8. Plot, Cursor und Marker

- **Plotbereiche**: Spalte *Plot* bzw. Tasten **1–4** legen markierte Variablen in Bereich
  1–4, **0** nimmt sie heraus. Alle Bereiche teilen die Zeitachse.
- **Zeitfenster** in der Plot-Leiste oder mit **+ / −**.
- **Zoomen**: links ziehen = Zoom-Rechteck, Mausrad (über einer Achse nur diese),
  rechts ziehen = Achsen strecken, Mitte ziehen = verschieben. Beim Zoomen bleibt die
  Ansicht stehen; **F5** bzw. **Leertaste** = live folgen / anhalten, **F** = ganze
  Messung, **Y** = Y-Achsen anpassen.
- **Cursor**: Klick in den Plot setzt Cursor 1; **,** und **.** schalten Cursor 1/2 ein.
  Die Cursorbox zeigt alle Kurven des Bereichs mit Werten an C1 und C2, Δy, Δt und 1/Δt.
- **Marker**: **N** setzt einen nummerierten Marker – im Plot und als Kanal *Marker* in
  der Aufzeichnung.
- Die ganze Messung bleibt im Plot (bis ca. 25 Mio. Messpunkte). Jede Spitze bleibt
  sichtbar; beim Hineinzoomen erscheinen die echten Messpunkte (**P**).

---

## 9. Aufzeichnen und Aufzeichnungen ansehen

- **M** (oder der Knopf mit rotem Punkt) startet/stoppt die Aufzeichnung – ohne Nachfrage,
  bei Bedarf wird erst verbunden. Datei: `Messung_JJJJMMTT_hhmmss.mf4` im Ablageordner
  (Standard: `Messungen\` im Projektordner).
- Format unter *Messung → Aufzeichnungsformat*: **MF4** (MDF4, CANape/asammdf; jedes
  Signal mit seinen echten Zeitpunkten) oder **CSV** (`;`-getrennt).
- *Messung → Aufzeichnen unter …* fragt nach Name und Ort; *Ablageordner …* ändert den
  Standardordner.
- **Ansehen**: *Datei → Aufzeichnung anzeigen …* (Strg+M) oder Datei ins Fenster ziehen.
  Auch fremde MF4- und CSV-Dateien (CSV: erste Spalte Zeit in s, Einheit aus
  `Name [Einheit]`). Cursor und Zoom wie live; F9 kehrt zur Live-Messung zurück.
- Optional *Messung → ECU-Zeitstempel verwenden*, wenn die A2L DAQ-Zeitstempel vorsieht.

---

## 10. Parameter schreiben

Schreiben ist nach jedem Start **gesperrt**.

1. *Ansicht → Schreiben freigeben* (**Strg+W**).
2. In der Spalte *Neuer Wert* tippen (oder **F2**), **Enter** schreibt.
3. QuickXCP prüft vorher: nur skalare Werte, nur Adressen, die laut Geräteprofil im
   **RAM** liegen; Flash, Peripherie und unbekannte Bereiche sind gesperrt. Warnungen
   bei Werten außerhalb `min`/`max`, 64-Bit-Werten (nicht atomar) und Bitfeldern.
4. Nach dem Schreiben wird zurückgelesen; jeder Schreibvorgang landet im **Schreib-Log**
   (*Werkzeuge → Ordner mit Schreib-Log/Sitzung öffnen*).

Es gibt keine Seitenumschaltung (Referenz-/Arbeitsseite): geschrieben wird direkt in den
RAM, nach einem Neustart des Steuergeräts gelten wieder die Werte aus dem Flash.

---

## 11. Tastenkürzel und Maus

Die vollständige, immer aktuelle Liste zeigt **F1** (*Hilfe → Tastenkürzel und Maus*).
Einzeltasten wirken nicht, während in einem Eingabefeld getippt wird.

| Taste | Funktion |
|---|---|
| **F9 / Esc** | Messung starten / stoppen und trennen |
| **M** | Aufzeichnung starten/stoppen |
| **N** | Marker setzen |
| **Strg+P** | Befehle und Variablen suchen |
| **Strg+F** | Variablensuche |
| **Strg+B** | Variablenbaum ein/aus |
| **Strg+O** | ELF öffnen |
| **Strg+M** | Aufzeichnung (MF4/CSV) anzeigen |
| **Strg+T** | neue Messseite |
| **Strg+Tab / Strg+Umschalt+Tab** | nächste / vorige Messseite |
| **Strg+Z / Strg+Y** | Messseite rückgängig / wiederholen |
| **Strg+W** | Schreiben freigeben / sperren |
| **F2** | neuen Wert für die markierte Variable eingeben |
| **Entf** | markierte Variablen entfernen |
| **Alt+↑ / Alt+↓** | Zeile verschieben |
| **1–4 / 0** | markierte Variablen in Plotbereich 1–4 / aus dem Plot |
| **H / D / B** | Anzeige Hex / Dezimal / Binär |
| **F5 / Leertaste** | Plot: live folgen / anhalten |
| **F** | Plot: ganze Messung |
| **Y** | Plot: Y-Achsen anpassen |
| **+ / −** | Zeitfenster halbieren / verdoppeln |
| **Strg++ / Strg+−** | Y-Achsen strecken / stauchen |
| **← → ↑ ↓** | Plot verschieben (nach Klick in den Plot) |
| **, / .** | Cursor 1 / Cursor 2 |
| **L / G / P** | Plot leeren / Raster / Messpunkte |
| **Strg+K / Strg+Umschalt+K** | Mini-Kurven in Tabelle / Anzeige |
| **Strg+Umschalt+A** | Tabelle ↔ große Anzeige |
| **Strg+0**, Strg+Mausrad | Zeilengröße der Tabelle |
| **F11 / Umschalt+F11** | Bereich maximieren / Vollbild |
| **Strg+D** | dunkles Design |
| **F1** | alle Tastenkürzel |

| Maus | Funktion |
|---|---|
| Klick in den Plot | Cursor setzen |
| links ziehen / Mitte ziehen / rechts ziehen | Zoom-Rechteck / verschieben / strecken |
| Mausrad im Plot | zoomen (über einer Achse nur diese) |
| Doppelklick im Variablenbaum | Variable in die Tabelle |
| Variable auf einen Plotbereich ziehen | in Tabelle und diesen Plotbereich |
| Datei ins Fenster ziehen | ELF, A2L, Aufzeichnung, Messseite oder Projekt öffnen |
| Titel eines Bereichs ziehen / doppelklicken | andocken / maximieren |

---

## 12. VS-Code-Erweiterung

Die Erweiterung verbindet **Visual Studio Code** mit einem laufenden QuickXCP: Variablen
direkt aus dem C-Code messen, Live-Werte neben dem Code sehen und Parameter schreiben. Sie
spricht nie selbst mit dem Steuergerät – Verbindung, Schreibsperre, Prüfungen und
Schreib-Log bleiben in QuickXCP.

### Installieren

- **Mit dem QuickXCP-Installer** (empfohlen): Option „Erweiterung für Visual Studio Code
  installieren“ angehakt lassen. Sie erscheint, wenn VS Code installiert ist. Updates von
  QuickXCP aktualisieren die Erweiterung mit.
- **Nachträglich / von Hand**: in einer Eingabeaufforderung

  ```
  code --install-extension "%LOCALAPPDATA%\Programs\QuickXCP\vscode\quickxcp-vscode.vsix" --force
  ```

  oder in VS Code: *Erweiterungen* → *…* → *Aus VSIX installieren …* und die Datei
  `quickxcp-vscode.vsix` wählen (liegt im Programmordner unter `vscode\` und am
  [Release](https://github.com/kaemmers/QuickXCP-releases/releases/latest)).
- VS Code danach neu laden. Für die Erkennung funktionsstatischer Variablen hilft ein
  C/C++-Sprachserver (z. B. die Microsoft-C/C++-Erweiterung).

### Benutzen

QuickXCP muss laufen und das passende ELF geladen haben. Die Statusleiste von VS Code
zeigt, ob QuickXCP läuft, ob das Steuergerät verbunden und ob Schreiben freigegeben ist.

| In einer C/C++-Datei | |
|---|---|
| **In QuickXCP messen** | Rechtsklick auf die Variable oder **Strg+Alt+M** – landet auf der aktuellen Messseite |
| **Wert beim Überfahren** | Hover zeigt aktuellen Wert, Typ und Adresse, mit Links zum Messen, Plotten und Schreiben |
| **Live-Werte im Editor** | alles, was gemessen wird, steht am Zeilenende, mit Mini-Verlauf (`▁▂▃▅▇`) |
| **Live-Plot** | Rechtsklick → *Im Live-Plot anzeigen* oder **Strg+Alt+P** – Panel „QuickXCP“ unten mit laufenden Kurven, Min/Max, Zeitfenster 10–120 s |
| **Wert schreiben** | Rechtsklick → *Wert in QuickXCP schreiben …* oder **Strg+Alt+W** – nur, wenn in QuickXCP freigegeben (Strg+W); dieselben Prüfungen und derselbe Warn-Dialog wie in QuickXCP |

Erkannt werden Ausdrücke wie `g_motor.speed` oder `arr[3].x`; Zugriffe über Zeiger
(`p->x`) nicht, weil die Adresse erst zur Laufzeit feststeht.

**Einstellungen** (VS Code → Einstellungen → „quickxcp“): `inlineValues`, `hover`,
`sparkline`, `sparklineSeconds`, `bridgeFile`.

**Verbindung:** QuickXCP öffnet beim Start eine Schnittstelle nur auf `127.0.0.1`
(zufälliger Port, Zugangstoken in `%APPDATA%\QuickXCP\bridge.json`). Mit der
Umgebungsvariable `QUICKXCP_NO_BRIDGE=1` startet QuickXCP ohne diese Schnittstelle.

---

## 13. Claude-Skill: Messseite aus Testplan

### Was der Skill kann

Nach einer Code-Änderung muss man beim Test wissen, **welche Variablen** man beobachten
und verstellen muss. Der Skill **„messseite-aus-testplan“** bringt dem KI-Assistenten
Claude bei, das zu erledigen:

- liest die **Code-Änderung** (Diff, Commit, Ticket) und den **Testplan**; gibt es keinen,
  leitet er die Testfälle aus der Änderung ab,
- sucht die messbaren Größen: **Eingänge**, die die geänderte Logik speisen, **Ausgänge,
  Zustände und Fehlerbits**, die das Ergebnis zeigen, und **Parameter**, mit denen man
  im Test stimuliert; für lokale Variablen nimmt er die globale Quelle,
- schlägt die **exakten Pfade im ELF** nach, statt sie zu raten,
- wählt **Polling oder DAQ-Event** (aus der A2L), **Plotbereiche** (Analogwerte, Flags
  und Zustände getrennt), Hex-Anzeige für Bitmasken und `min`/`max` für Parameter,
- schreibt eine importierbare **`.qxpage`**, **prüft** sie gegen ELF und A2L und
  korrigiert, bis keine Fehler mehr kommen,
- liefert eine Tabelle **Variable | Rolle | Warum | Erwartung im Test** und sagt, was
  nicht messbar ist.

Der Skill erstellt nur die Datei – er verbindet sich nie mit dem Steuergerät und schreibt
keine Werte.

### Installieren

Der Skill liegt im Programmordner unter `skills\messseite-aus-testplan\` und als ZIP am
[Release](https://github.com/kaemmers/QuickXCP-releases/releases/latest/download/messseite-aus-testplan.zip).

- **Claude Code** (empfohlen, weil Claude dort die Prüfung selbst ausführen kann):
  Ordner nach `%USERPROFILE%\.claude\skills\` kopieren (gilt für alle Projekte) oder
  nach `.claude\skills\` im Repository der Firmware (gilt für dieses Projekt):

  ```powershell
  Copy-Item -Recurse "$env:LOCALAPPDATA\Programs\QuickXCP\skills\messseite-aus-testplan" "$env:USERPROFILE\.claude\skills\"
  ```

- **claude.ai / Claude Desktop**: *Einstellungen → Fähigkeiten (Skills)* → die ZIP-Datei
  hochladen. Dort kann Claude `quickxcp-cli` meist nicht ausführen; die Seite ist dann
  ungeprüft (Claude sagt das dazu) und wird beim Import in QuickXCP geprüft – unbekannte
  Variablen erscheinen ausgegraut.

### Anwenden

In Claude Code im Repository der Firmware, z. B.:

> Erstelle mir eine Messseite für den Test von Commit `abc1234`. Testplan liegt in
> `docs/testplan_drehzahl.md`, ELF ist `build/app.elf`, A2L `build/app.a2l`.

oder einfach: „Welche Variablen brauche ich zum Testen dieser Änderung? Mach mir eine
QuickXCP-Messseite daraus.“ Claude erkennt den Skill an solchen Formulierungen
automatisch; ausdrücklich geht es mit `/messseite-aus-testplan`.

Ergebnis: eine `.qxpage`-Datei plus Begründungstabelle. In QuickXCP die Datei **ins
Fenster ziehen** (oder Rechtsklick auf einen Reiter → *Importieren …*), F9 – fertig.

### Prüfbefehl (auch ohne Claude nutzbar)

```
quickxcp-cli page firmware.elf seite.qxpage --a2l projekt.a2l --write
```

- meldet unbekannte Pfade mit Vorschlägen („meinten Sie: g_motor.speed …“) und
  Event-Kanäle, die es in der A2L nicht gibt (Exit-Code 1 bei Fehlern),
- `--write`: löst Strukturen und Arrays in ihre Elemente auf, entfernt Doppelte,
  korrigiert ungültige Werte und speichert im Standardformat (`-o` = andere Zieldatei).

---

## 14. Kommandozeile (`quickxcp-cli.exe`)

Liegt im Programmordner (`%LOCALAPPDATA%\Programs\QuickXCP\`).

| Befehl | Zweck |
|---|---|
| `quickxcp-cli vars app.elf "motor speed"` | Variablen suchen (Adresse, Größe, Typ, Pfad) |
| `quickxcp-cli elfcheck app.elf` | Prüfbericht: ist das DWARF vollständig? |
| `quickxcp-cli a2l app.a2l` | Verbindungsdaten und Events aus der A2L |
| `quickxcp-cli page app.elf seite.qxpage --a2l app.a2l` | Messseite prüfen (siehe [13](#prüfbefehl-auch-ohne-claude-nutzbar)) |
| `quickxcp-cli read app.elf g_motor.speed --iface profile --a2l app.a2l` | Werte einmalig lesen |
| `quickxcp-cli diag --vn-channel 1 --a2l app.a2l` | CAN/XCP-Diagnose am Vector-Interface |
| `quickxcp-cli diag --scan --bitrate all` | alle Vector-Kanäle und Standard-Bitraten nach XCP absuchen |

Gemeinsame Optionen: `--profile datei.toml` (Geräteprofil), `--bitrate 500k`,
`--iface sim|profile|vector|pcan|kvaser|ixxat`, `--channel`, `--vn-channel`, `-v`.

---

## 15. Dateien und Ablageorte

| Was | Wo |
|---|---|
| Programm, `quickxcp-cli.exe`, Handbuch, Skill, VS-Code-Erweiterung | `%LOCALAPPDATA%\Programs\QuickXCP\` |
| Vector-DLL (optional) | `…\QuickXCP\vector\vxlapi64.dll` |
| Geräteprofile | `%APPDATA%\QuickXCP\profiles\*.toml` oder `profiles\` neben der exe |
| Persönliche Einstellungen | `%APPDATA%\QuickXCP\session.json` |
| Standardprojekt | `%APPDATA%\QuickXCP\Projekt\` |
| Schreib-Log, Programm-Log (`quickxcp.log`) | `%APPDATA%\QuickXCP\` |
| Verbindungsdaten für VS Code | `%APPDATA%\QuickXCP\bridge.json` |
| ELF-Cache (darf gelöscht werden) | `%LOCALAPPDATA%\QuickXCP\cache\` |
| Projekte, Messseiten, Aufzeichnungen | frei wählbar (Projektordner, `Messungen\`) |

---

## 16. Fehlersuche

| Problem | Lösung |
|---|---|
| CAN-Interface erscheint nicht | Herstellertreiber aktualisieren; unter Windows 11 Speicher-Integrität prüfen. Vector: `vxlapi64.dll` installiert oder in `vector\`? |
| Verbinden schlägt fehl (Timeout) | Kanal und Bitrate prüfen (Statusleiste), Steuergerät an? `quickxcp-cli diag --scan --bitrate all` sucht XCP auf allen Kanälen. CANape beendet? |
| Warnung „anderer XCP-Master“ | Ein anderes Programm spricht auf denselben IDs – beenden. |
| Warnung beim ELF-Abgleich | Das ELF passt nicht zur Software im Steuergerät: passendes ELF laden oder flashen. |
| Variable ausgegraut („nicht im ELF“) | Im geladenen ELF gibt es sie nicht (umbenannt, entfernt, `#ifdef`). Passendes ELF laden oder mit Entf entfernen. |
| DAQ-Variable bleibt bei „–“ | Event in der A2L prüfen, Buslast prüfen; zum Test auf Polling stellen. |
| Schreiben verweigert | Strg+W gedrückt? Adresse im Geräteprofil als `ram` eingetragen? |
| Update lässt sich nicht installieren | QuickXCP-Fenster schließen, Installer von der Release-Seite von Hand starten. |
| Sonstiges | Log-Bereich im Fenster und `%APPDATA%\QuickXCP\quickxcp.log` ansehen. |

---

## 17. Änderungshistorie

### 0.2.2

- **VS-Code-Erweiterung**: Variablen im C-Code messen (Strg+Alt+M), Wert im Hover,
  Live-Werte und Mini-Verlauf am Zeilenende, Live-Plot-Panel (Strg+Alt+P), Parameter
  schreiben (Strg+Alt+W, nur bei Freigabe in QuickXCP). Wird vom Installer optional
  mitinstalliert und mit aktualisiert.
- **Claude-Skill „messseite-aus-testplan“**: stellt aus Code-Änderung und Testplan eine
  Messseite zusammen; liegt im Programmordner und als ZIP am Release.
- **Neuer Befehl `quickxcp-cli page`**: Messseite gegen ELF und A2L prüfen, mit
  Vorschlägen bei Tippfehlern; Strukturen auflösen.
- **Oberfläche aufgefrischt**: Bereiche als Karten, neue Symbole, Startansicht mit
  Drag & Drop; **Befehls- und Variablensuche** (Strg+P); Variablenbaum standardmäßig
  ausgeblendet (Strg+B).
- **Mini-Kurven** in Tabelle und Anzeige (Strg+K / Strg+Umschalt+K).
- **Messtabelle**: Strg+Mausrad für Schrift- und Zeilengröße, Spalten verschiebbar und
  ein-/ausblendbar, Einstellungen werden gemerkt.
- **Plot**: Legende links oben, dünne Cursorlinie zum Ziehen, gleich hohe Bereiche, Zoom
  aus der Pause bleibt erhalten; schneller unter Last.
- **Warnung, wenn ein anderer XCP-Master** auf denselben CAN-IDs sendet.
- DAQ-Daten werden schneller dekodiert.
- Behoben: Cursorbox zeigte nach einem Seitenwechsel ohne Daten noch die alte Seite.
- Dieses Handbuch.

### 0.2.1

- Messseiten-Reiter direkt am Inhaltsbereich; Anordnung der Bereiche je Messseite.
- **Große Anzeige** (Kacheln) als Alternative zur Tabelle, Vollbild Umschalt+F11.
- Zeilen und Kacheln per Ziehen umsortieren, Tabelle auch mit Alt+Pfeil.
- Plot: Klick wählt die Kurve für den Cursor; Cursorbox mit allen Kurven des Bereichs,
  C1, C2, Δy und Δt; *live folgen* und *Pause* in einem Knopf (Leertaste/F5).
- **Messseite während der Messung wechseln** (gesperrt nur bei Aufzeichnung).
- Behoben: Nach einem Timeout konnten XCP-Antworten verrutschen (falsche Werte, Fehler beim
  DAQ-Neuaufsetzen). QuickXCP resynchronisiert jetzt automatisch.

### 0.2.0

Erste verteilte Version mit Installer und **automatischen Updates**. Enthält den
Grundumfang: Variablen aus ELF/DWARF, Verbindungsdaten aus der A2L, Polling und Dynamic
DAQ, Live-Plot mit bis zu vier Bereichen, Cursorn und Markern, Aufzeichnung als MDF4/CSV
und deren Anzeige, A2L-Umrechnungen (physikalische Werte, Einheiten, Texttabellen),
Parameter schreiben mit Schreibschutz und Schreib-Log, Geräteprofile, Projekte und
Messseiten, ELF/Firmware-Abgleich, Buslastanzeige, Wiederverbinden nach
Steuergeräte-Neustart, frei andockbare Bereiche, helles/dunkles Design, Simulator und
Kommandozeile.
