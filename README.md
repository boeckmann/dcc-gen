# DCC Signal Generator

DCC (Digital Command Control) signal generator for Arduino Uno R3 and
compatible devices with Arduino Motor Shield for operating digital model
railways based on the standards of https://railcommunity.de, in particular
the RCN-210, RCN-211, and RCN-212.

Implemented features:

- Timing-correct generation of DCC packets
- Short (1-127) and long locomotive addresses (1-10239)
- Functions 0-68
- 14, 28, and 128 speed steps
- Emergency stop and idle packets

Planned features:

- Emergency shutdown of track power in case of a short circuit
- Continuous feedback mode via serial interface for monitoring the load
- Programming mode

Limitations:

- Approximately 80 locomotives can be operated simultaneously with one Arduino
  Uno R3 (ATmega328) (memory limit). This value is rather theoretical,
  however, as the Arduino motor shields with L298 H-bridges can only drive a
  maximum of 2A. Realistically, this corresponds to approximately 8-10
  trains.

## Command Overview

The DCC generator accepts text-based commands for control via a serial
interface (USB). In the simplest case, a program for controlling the serial
interface (e.g., minicom, Hterm) is sufficient for testing. There is an
approximately two-second timeout between individual characters when reading a
command. Therefore, you should not take too long when typing commands
manually.

Commands begin with `!` and end with a line break.

After successful execution of a command, the program sends `?OK` and a line
break.

If an error occurs, it sends `?ERR` and a line break.

The following commands are accepted, where z represents a valid locomotive
address and n represents a valid number appropriate to the context:

```
!I     Retrieve version information

!R     disable all trains and reset speed and functions,
       send DCC reset packets  
!S     reset speed and functions of all trains,
       NO DCC reset packets are sent

!P+    Enable track power
!P-    Disable track power
!H+    Enable emergency stop for all trains
!H-    Disable emergency stop

!zA+   Enable DCC signal for train z
!zA-   Disable DCC signal for train z

!zC+   Set 14 speed steps for train z
!zC0   Set 14 speed steps
!zC-   Set 28 speed steps for train z (default)
!zC1   Set 28 speed steps
!zC2   Set 128 speed steps for train z

!zVn   Set forward speed n for train z
!zRn   Set reverse speed n for train z Set

!zFn+  Activate function n for train z
!zFn-  Deactivate function n for train z
```

In the following example, track power and signal generation for locomotive
address 1 are activated, the headlights are switched on (function 0), and
speed level 14 (forward) is set. Provided everything is wired correctly, a
DCC-compliant locomotive should then start moving.

```
!P+
!1A+
!1F0+
!1V14
```

## Packet Scheduling

The generator continuously outputs a DCC signal. If no locomotive address is
active, DCC idle packets are output to maintain power supply. If locomotive
addresses are active, the packets for speed and functions 0-12 are repeated
at regular intervals. This occurs alternately for all active locomotive
addresses, meaning they are all treated equally. When changes to the speed
steps or functions are received via the serial interface (USB), the resulting
DCC packets are sent out of sequence and repeated several times. The goal is
for the receiving locomotive to receive the information a) as quickly as
possible and b) as reliably as possible.

Functions 0-12 of each activated locomotive address are sent continuously. For
optimization reasons, this differs for functions 13-68. Continuous sending of
these functions for a specific locomotive address is only activated after a
set or reset command for one of these functions has been received.

## Note regarding the Arduino Motor Shield

The DCC standards specify a track voltage of approximately 18V
(max.), depending on the track gauge. This voltage can be supplied to the
motor shield via the Vin and GND input terminals using a suitable power
supply. CAUTION: The connection to the Arduino's Vin port must be
disconnected beforehand. Some motor shields have a jumper wire for this
purpose. Failure to do so will destroy the Arduino!

## Building and Uploading the Program

AVR-GCC is required for building. I tested it with GCC 11 and GCC 13. Other
versions should also work. For GCC 12-13.2, the note in the Makefile must be
observed! To build the program, with a correctly configured AVR-GCC
toolchain, a simple `make` in the project directory should suffice. The
program can be uploaded to the Arduino with `make upload PORT=... BAUD=...`.
This requires an installed avrdude program. PORT and BAUD(default 115200)
must be set.


# DCC Signalgenerator (german)

DCC (Digital Command Control) Signalgenerator für Arduino Uno R3 und
kompatible Geräte mit Arduino Motor-Shield zum Betrieb von digitalen
Modelleisenbahnen basierend auf den Normen der https://railcommunity.de,
insbesondere den Normen RCN-210, RCN-211 und RCN-212.

Umgesetzte Funktionen:

- Timing-Korrekte Generierung von DCC Paketen
- Kurze (1-127) und lange Lokadressen (1-10239)
- Funktionen 0-68
- 14, 28 und 128 Fahrstufen
- Nothalt- und Idle-Pakete

Geplant:

- Notabschaltung Gleisspeisung bei Kurzschluss
- Kontinuierlicher Rückmeldemodus über Serielle Schnittstelle zur Überwachung
  der Last
- Programmiermodus

Limitierungen:

- Mit einem Arduino Uno R3 (ATmega328) können ca. 80 Loks gleichzeitig
  betrieben werden (Arbeitsspeicherlimit). Dieser Wert ist aber eher
  theoretischer Natur, da die Arduino Motor-Shields mit L298 H-Brücke maximal
  2A treiben können. Realistischer Weise entspricht das ca. 8-10 Zügen.

## Kommandoübersicht

Der DCC Generator nimmt über serielle Schnittstelle (USB) textbasierte
Kommandos zur Steuerung entgegen. Im einfachsten Fall reicht zum Testen also
ein Programm zur Ansteuerung der Seriellen Schnittstelle(minicom, Hterm
etc.). Es existiert ein ca. zwei Sekunden Timeout zwischen einzelnen Zeichen
beim Lesen eines Kommandos. Beim händischen Tippen von Kommandos sollte man
sich daher nicht all zu viel Zeit lassen.

Kommandos beginnen mit `!` und enden mit Zeilenvorschub. Nach erfolgreicher
Abarbeitung eines Kommandos sendet das Programm `?OK` und Zeilenvorschub. Bei
auftreten eines Fehlers `?ERR` und Zeilenvorschub.

Folgende Kommandos werden akzeptiert, dabei steht z für eine gültige
Lokadresse und n für eine dem Kontext entsprechend gültige Zahl:

```
!I        Versionsinformation abrufen

!R        Reset: DCC-Reset Pakete senden,
          alle Züge deaktivieren und Geschwindigkeit sowie Funktionen zurücksetzen
!S        Geschwindigkeit und Funktionen aller Züge zurücksetzen, ohne Reset

!P+       Gleisspeisung ein
!P-       Gleisspeisung aus
!H+       Nothalt für alle Züge ein
!H-       Nothalt aus

!zA+      DCC Signal Zug z aktivieren
!zA-      DCC Signal Zug z deaktivieren

!zC+      14 Fahrstufen-Modus für Zug z
!zC0      14 Fahrstufen-Modus
!zC-      28 Fahrstufen-Modus für Zug z (standard)
!zC1      28 Fahrstufen-Modus
!zC2      128 Fahrstufen für Zug z

!zVn      Vorwärtsgeschwindigkeit n für Zug z setzen
!zRn      Rückwärtsgeschwindigkeit n für Zug z setzen

!zFn+     Funktion n für Zug z aktivieren
!zFn-     Funktion n für Zug z deaktivieren
```


Im folgenden Beispiel wird die Gleisspeisung sowie die Generierung des Signals
für Lokadresse 1 aktiviert, das Fahrtlicht eingeschaltet (Funktion 0) und die
Geschwindigkeitsstufe 14 (vorwärts) gesetzt. Sofern alles elektrisch richtig
verdrahtet ist sollte sich eine DCC konforme Lok daraufhin in Bewegung
setzen.

```
!P+
!1A+
!1F0+
!1V14
```

## Paket-Scheduling

Der Generator gibt kontinuierlich ein DCC Signal aus. Sollte keine Lokadresse
aktiv sein werden zur Aufrechterhaltung der Stromversorgung DCC Idle-Pakete
ausgegeben. Sollten Lokadressen aktiv sein werden die Pakete für
Geschwindigkeit und Funktionen 0-12 in regelmäßigen Abständen wiederholt.
Dies geschieht für alle aktiven Lokadressen alternierend, also
gleichberechtigt. Wenn Änderungen an den Fahrstufen oder Funktionen mittels
serieller Schnittstelle (USB) eingehen, werden die daraus resultierenden DCC
Pakete außer der Reihe und mehrmals wiederholt gesendet. Ziel ist, dass die
Empfängerlok die Information a) so schnell wie möglich und b) möglichst
zuverlässig empfängt.

Die Funktionen 0-12 jeder aktivierten Lokadresse werden kontinuierlich
gesendet. Dies ist bei den Funktionen 13-68 aus Optimierungsgründen anders.
Das kontinuierliche Senden dieser Funktionen für eine spezifische Lokadresse
wird nur aktiviert, nachdem ein Kommando zum Setzen oder Rücksetzen für eine
dieser Funktionen eingegangen ist.


## Hinweis zum Arduino Motor-Shield

In den DCC Normen ist als Gleisspannung ein Wert von (je nach Spurgröße) ca.
18V (max.) spezifiziert. Diese Spannung kann dem Motor-Shield mittels
geeignetem Netzteil über die Eingangsklemmen Vin und GND zugeführt werden.
ACHTUNG: Die Leitung zum Vin Port des Arduino muss unbedingt vorher gekappt
werden. Bei einigen Motor-Shields existiert dafür eine Drahtbrücke. Erfolgt
dies nicht führt dies zur Zerstörung des Arduino!


## Bauen und Hochladen des Programms
Es wird AVR-GCC zum Bauen benötigt. Getestet habe ich es mit GCC 11 und
GCC 13. Andere Versionen sollten ebenfalls funktionieren. Für GCC 12-13.2
muss der Hinweis im Makefile beachtet werden! Zum Bauen des Programms sollte
bei korrekt eingerichtetem ACR-GCC Toolchain ein simples `make` im
Projektverzeichnis genügen. Das Programm kann mit `make upload PORT=... BAUD=...`
auf den Arduino geladen werden. Dies setzt ein installiertes avrdude Programm
voraus. PORT und BAUD (standardmäßig 115200) müssen entsprechend der Seriellen
Schnittstelle angepasst werden.
