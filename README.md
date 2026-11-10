# Pong2090

Pong für den Busch Microtronic 2090 mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) (8x8-LED-Matrix) und 2 Tastern: der Spieler links gegen den Computer rechts in 15 Spielstufen, oder allein gegen die Wand.

- 213 Programmschritte, 43 frei
- Dateien: [pong.mic](pong.mic) (Version 02), die Quelle [pong.src](pong.src) mit Sprungmarken und die Arbeitsbeschreibung [pong.xml](pong.xml)

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

## Wie es gemacht ist

Der 2090 rechnet nur mit ganzen Zahlen von 0 bis 15, kennt keine indirekte Adressierung und merkt sich nur eine Rücksprungadresse. Ein paar Kniffe, mit denen Pong trotzdem in 213 Schritte passt:

### Schräge Bahnen ohne Brüche: die Idee von Bresenham

Der Ball fliegt je Schritt genau eine Spalte weiter. Für die steile Bahn steigt er dabei eine ganze Reihe, für die flache nur eine halbe. Halbe Reihen kann die Matrix nicht zeigen, und Brüche kennt der 2090 nicht.

Die Lösung ist dieselbe Idee, mit der Bresenhams Algorithmus schräge Linien auf ein Raster bringt: Man zählt den Rest mit, statt ihn wegzuwerfen. Die Höhe des Balls steht in halben Reihen in einem Register (BY, 0 bis 15). Je Schritt kommt 1 (flach) oder 2 (steil) dazu, abwärts F oder E, das ist -1 und -2. Angezeigt wird die Reihe BY / 2, also ein `SHR`. Das unterste Bit ist der Rest, der „Fehler“ in Bresenhams Sprache: Bei der flachen Bahn kippt es bei jedem Schritt, und jedes zweite Mal rückt der Ball dadurch eine Reihe weiter. So entsteht die Treppe 1 Reihe je 2 Spalten, ganz ohne Vergleich oder Sonderfall.

Das ist Bresenham in seiner einfachsten Form, mit nur zwei möglichen Steigungen. Den vollständigen Algorithmus für beliebige Linien hat übrigens die Matrix selbst eingebaut (Zeichenbefehl ACK 8), Pong braucht ihn aber nicht.

Der Abprall oben und unten nutzt den Überlauf: Läuft BY beim Addieren über 15 oder unter 0, zeigt das der Carry. Dann wird die Höhe gespiegelt (`INV`) und die Richtung umgedreht.

### Die Spielstärke des Computers: noch einmal dieselbe Idee

Der Computer soll je nach Stufe in 1 bis 15 von 16 Ballschritten ziehen dürfen, und zwar gleichmäßig verteilt, nicht in Schüben. Dafür addiert das Programm bei jedem Schritt die Stufe auf einen Zähler (`ADD AS,AA`). Läuft er über, darf der Computer ziehen. Bei Stufe 8 ist das jeder zweite Schritt, bei Stufe 12 drei von vier, bei Stufe 3 etwa jeder fünfte. Auch das ist Bresenhams Prinzip: einen Bruch (Stufe/16) durch Aufsummieren und Überlauf gleichmäßig verteilen. Ein Würfel wäre teurer und ungleichmäßiger.

Der Computer zieht nur, wenn der Ball auf ihn zufliegt und über oder unter seinem Schläger ist. Dadurch kann er nie über den Rand hinaus, eine Randprüfung entfällt.

### Der Treffer: drei Fälle aus einer Subtraktion

Trifft der Ball die Schlägerspalte, rechnet das Programm V = Reihe des Balls minus unterste Reihe des Schlägers. V ist 0, 1 oder 2 für unten, Mitte, oben. Alles andere ist daneben, auch negative Werte, denn die erscheinen im 4-Bit-Register als 11 bis 15. Ein einziger Vergleich (`CMPI 2`) trennt also Treffer und Fehler.

Aus V wird dann die neue Richtung: (V - 1) * 2 ergibt -2, 0 oder 2, also steil abwärts, Mitte, steil aufwärts. Bei der Mitte bleibt die bisherige senkrechte Richtung, nur flach. Das Vorzeichen holt sich das Programm ohne Sprung aus dem Carry eines Vergleichs.

### Zwei Befehle direkt hintereinander

Ein Zeichenbefehl an die Matrix ist: auf REQ warten, ACK, zwei Nibbles, Ausgänge auf 0. Für eine Bewegung braucht es zwei davon. Die Routine `move` sendet den ersten (alter Punkt aus) und läuft dann einfach in die Routine `cmd` für den zweiten (neuer Punkt an) hinein. So gibt es den Code nur einmal, und zwischen den beiden Befehlen vergeht keine Zeit für Rücksprung und neuen Aufruf.

Ob ein Punkt an- oder ausgeschaltet wird, steckt in den Koordinaten selbst: y + 8 heißt „Pixel an“, x + 8 und y + 8 heißt „Pixel aus“. Die Farbe bleibt in beiden Fällen, wie sie in der Farbmatrix steht.

### Unterprogramm im Unterprogramm, obwohl es das nicht gibt

Der 2090 merkt sich nur eine Rücksprungadresse. Ein Unterprogramm darf deshalb kein weiteres aufrufen. Die Tasterabfrage `ply` ist ein Unterprogramm und muss trotzdem `move` benutzen. Der Ausweg: Sie springt mit `GOTO` nach `move`, und dessen `RET` kehrt direkt zum Aufrufer von `ply` zurück. So kann `ply` zweimal je Ballschritt aufgerufen werden, ohne doppelt im Speicher zu stehen.

### Der Spielstand in der zweiten Registerbank

Alle 16 Register sind im Spiel belegt. Die Punkte liegen deshalb in den Speicherregistern M6 und M7. Fällt ein Punkt, tauscht `EXRL` die untere Registerbank gegen die Speicherregister, das Programm zählt und zeigt den Stand an, und ein zweites `EXRL` holt die Spielregister zurück. `EXRL` verändert die Flags nicht. Der Vergleich „9 erreicht?“ gilt also noch nach dem Tausch.

### Ein Register, das immer 0 ist

Nach jedem Befehl müssen die Ausgänge auf 0. R0 wird im ganzen Programm nie beschrieben und ist nach dem Einschalten 0. Ein `DOT` von R0 spart jedes Mal das vorherige `MOVI 0`.

Die `.mic` wird mit dem Mini-Assembler `micasm.py` aus `pong.src` erzeugt.
