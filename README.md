# Pong2090

Pong für den Busch Microtronic 2090 mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) (8x8-LED-Matrix) und 2 Tastern: der Spieler links gegen den Computer rechts in 15 Spielstufen, oder allein gegen die Wand.

- 213 Programmschritte, 43 frei
- Dateien: [pong.mic](pong.mic) (Version 02) und die Quelle [pong.src](pong.src) mit Sprungmarken

## Spiel

- **Spielstufe:** die erste Hex-Taste nach RUN. 1-F spielt gegen den Computer, je höher, desto stärker. 0 ist das Spiel allein gegen die Wand.
- **Schläger:** je 3 LEDs hoch und grün, der eigene in Spalte 1, der des Computers in Spalte 8.
- **Ball:** gelb.
- **Winkel:** Ein Treffer mit der Schlägermitte spielt flach zurück. Mit der oberen oder unteren Kante geht der Ball steil zurück, weg von der Mitte. Oben und unten prallt der Ball ab.
- **Punkt:** Die Anzeige zeigt Spieler und Computer. Mit einer Hex-Taste geht es weiter.
- **Sieg:** Wer zuerst 9 Punkte hat, gewinnt.
- **Wand (Stufe 0):** Der erste Fehler beendet das Spiel. Die Anzeige zeigt die Treffer hexadezimal.

## Bedienung

| Taster | Pfeiltaste im Emulator | Wirkung |
|---|---|---|
| Eingang 2 | hoch | Schläger hoch |
| Eingang 3 | runter | Schläger runter |

Die Taster werden zweimal je Ballschritt gelesen. Der eigene Schläger kann also zwei Reihen ziehen, während der Ball eine Spalte weiterfliegt.

## Start

RESET, Matrix einschalten („LED on“ in der Weboberfläche, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) Taste `m`), `HALT` `NEXT` `00` `RUN`, dann die Spielstufe eingeben. Neues Spiel: RESET, RUN.

## Technik: Zeichenbefehle statt ganzer Bilder

Version 01 schickte nach jedem Schritt das ganze Bild (16 Register). Version 02 ist neu geschrieben und sendet nur, was sich ändert:

- **Die Matrix merkt sich das Bild.** Das Programm braucht keinen Bildspeicher mehr, nur noch die Lage von Ball und Schlägern.
- **Bewegung = zwei Punkte.** Bewegt sich der Ball oder ein Schläger um ein Feld, schaltet das Programm den alten Punkt aus und den neuen an (ACK 4, zwei Befehle direkt hintereinander).
- **Farben ohne Umschalten.** Am Anfang setzt das Programm einmal die Farbmatrix: alles grün, Spalte 2-7 gelb. Danach schalten die Befehle nur noch Pixel an und aus, die Farbe kommt aus der Farbmatrix. Der Ball ist deshalb gelb und die Schläger grün, ohne dass die Farbe im Spiel je gewechselt wird.
- **Matrix dauernd aktiv.** Die Taster liegen an Eingang 2 und 3, Eingang 1 bleibt für die Anfrage der Matrix (REQ). Schlafen und Wecken braucht das Programm nicht.

Im Emulator dauert ein Ballschritt im Mittel 0,66 s (Version 01: 0,95 s), und die Taster werden etwa alle 0,3 s gelesen statt einmal je Schritt.

Die `.mic` wird mit dem Mini-Assembler `micasm.py` aus `pong.src` erzeugt.
