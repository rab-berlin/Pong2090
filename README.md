# Pong2090

Pong für den Busch Microtronic 2090 mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) (8x8-LED-Matrix) und 2 Tastern: der Spieler links gegen den Computer rechts in 15 Spielstufen, oder allein gegen die Wand.

- 237 Programmschritte, 19 frei
- Dateien: [pong.mic](pong.mic) (Version 03) und die Arbeitsbeschreibung [pong.xml](pong.xml)

## Spiel

- **Spielstufe:** die erste Hex-Taste nach RUN. 1-F spielt gegen den Computer, je höher, desto weniger Fehler macht er. 0 ist das Spiel allein gegen die Wand.
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

## Geschichte

- **Magnavox Odyssey:** Das Tischtennis-Prinzip steckt schon in der ersten Heimspielkonsole. Ralph Baer leitete das Team bei Sanders Associates, Bill Rusch skizzierte das Ping-Pong-Spiel in einem Memo vom 18. Oktober 1967. Die Odyssey hatte weder Ton noch Punkteanzeige, und die Höhe des Balls stellte man mit einem „English“-Knopf ein.
- **Nolan Bushnell** sah die Odyssey am 24. Mai 1972 bei einer Vorführung. Er stellte im Juni 1972 **Allan Alcorn** (24, vorher Ampex) ein und gab ihm als Aufwärmübung ein Ball-und-Schläger-Spiel mit Punktestand. Es war als Übung gedacht. Alcorn und Ted Dabney fanden es so gut, dass daraus ein Produkt wurde.
- **Alcorns Verbesserungen:** Jeder Schläger ist in **acht Segmente** geteilt, und der Abprallwinkel hängt vom getroffenen Segment ab. Bei langen Ballwechseln wird der Ball schneller, und nach einem Punkt beginnt er wieder langsam. Oben und unten prallt der Ball an Wänden ab. Dazu gab es einen einfachen Ton für Schläger und Wand, den „Pong“.
- **Andy Capp's Tavern:** Der Prototyp stand in dieser Bar, neben dem Automaten *Computer Space*. Es sprach sich herum, und morgens um 9 Uhr standen die Leute an. Nach etwa zwei Wochen rief die Bar Alcorn nachts an, weil der Automat nicht mehr ging. Als er die Münzbox öffnete, fielen ihm die Vierteldollarstücke entgegen: Sie war übergequollen und hatte den Mechanismus blockiert.
- **November 1972:** Atari brachte *Pong* heraus.
- Dieses Programm übernimmt die Idee, dass der Treffpunkt auf dem Schläger den Winkel bestimmt, aber nur mit zwei Stufen (flach oder steil) statt acht Segmenten.

Quelle: Benj Edwards, „The Making of Pong“, Vintage Computing and Gaming (2012).

## Technik: Zeichenbefehle statt ganzer Bilder

Version 01 schickte nach jedem Schritt das ganze Bild (16 Register). Seit Version 02 ist das Programm neu geschrieben und sendet nur, was sich ändert:

- **Die Matrix merkt sich das Bild.** Das Programm braucht keinen Bildspeicher mehr, nur noch die Lage von Ball und Schlägern.
- **Bewegung = zwei Punkte.** Bewegt sich der Ball oder ein Schläger um ein Feld, schaltet das Programm den alten Punkt aus und den neuen an (ACK 4, zwei Befehle direkt hintereinander).
- **Farben ohne Umschalten.** Am Anfang setzt das Programm einmal die Farbmatrix: alles grün, Spalte 2-7 gelb. Danach schalten die Befehle nur noch Pixel an und aus, die Farbe kommt aus der Farbmatrix. Der Ball ist deshalb gelb und die Schläger grün, ohne dass die Farbe im Spiel je gewechselt wird.
- **Matrix dauernd aktiv.** Die Taster liegen an Eingang 2 und 3, Eingang 1 bleibt für die Anfrage der Matrix (REQ). Schlafen und Wecken braucht das Programm nicht.

Im Emulator dauert ein Ballschritt im Mittel 0,82 s (Version 01: 0,95 s), und die Taster werden etwa alle 0,4 s gelesen statt einmal je Schritt. Ein gutes Stück der Zeit kostet der Computergegner, der fast bei jedem Schritt zieht.

## Wie es gemacht ist

Der 2090 rechnet nur mit ganzen Zahlen von 0 bis 15, kennt keine indirekte Adressierung und merkt sich nur eine Rücksprungadresse. Ein paar Kniffe, mit denen Pong trotzdem in 237 Schritte passt:

### Schräge Bahnen ohne Brüche: die Idee von Bresenham

Der Ball fliegt je Schritt genau eine Spalte weiter. Für die steile Bahn steigt er dabei eine ganze Reihe, für die flache nur eine halbe. Halbe Reihen kann die Matrix nicht zeigen, und Brüche kennt der 2090 nicht.

Die Lösung ist dieselbe Idee, mit der Bresenhams Algorithmus schräge Linien auf ein Raster bringt: Man zählt den Rest mit, statt ihn wegzuwerfen. Die Höhe des Balls steht in halben Reihen in einem Register (BY, 0 bis 15). Je Schritt kommt 1 (flach) oder 2 (steil) dazu, abwärts F oder E, das ist -1 und -2. Angezeigt wird die Reihe BY / 2, also ein `SHR`. Das unterste Bit ist der Rest, der „Fehler“ in Bresenhams Sprache: Bei der flachen Bahn kippt es bei jedem Schritt, und jedes zweite Mal rückt der Ball dadurch eine Reihe weiter. So entsteht die Treppe 1 Reihe je 2 Spalten, ganz ohne Vergleich oder Sonderfall.

Das ist Bresenham in seiner einfachsten Form, mit nur zwei möglichen Steigungen. Den vollständigen Algorithmus für beliebige Linien hat übrigens die Matrix selbst eingebaut (Zeichenbefehl ACK 8), Pong braucht ihn aber nicht.

Der Abprall oben und unten nutzt den Überlauf: Läuft BY beim Addieren über 15 oder unter 0, zeigt das der Carry. Dann wird die Höhe gespiegelt (`INV`) und die Richtung umgedreht.

### Ein Gegner, der Fehler macht

Ein Computergegner, der immer richtig zieht, ist langweilig, und einer, der nur selten ziehen darf, ist kein Gegner. Der Computer spielt deshalb wie ein Mensch: Er will bei jedem Ballschritt das Richtige tun, ist aber nicht immer bei der Sache.

- **Was er will:** Fliegt der Ball auf ihn zu, zieht er eine Reihe zum Ball, sobald der über oder unter seinem Schläger ist. Fliegt der Ball weg, schaut er auf den Gegner: In einem von vier Schritten geht er dem Schläger des Spielers nach. So steht er nie starr herum.
- **Wie konzentriert er ist:** Bei jedem Schritt würfelt das Programm (`RND`). Die Spielstufe bestimmt, wie oft er unkonzentriert ist: bei Stufe 1 in knapp der Hälfte der Schritte, bei Stufe 8 in einem Viertel, bei Stufe F fast nie.
- **Was dann passiert:** Unkonzentriert zögert er in der Hälfte der Fälle, also kein Zug. In einem Viertel zieht er in die falsche Richtung, und in einem Viertel macht er es trotzdem richtig.

Auch auf Stufe 1 trifft er damit die meisten Bälle, verliert aber gegen einen aufmerksamen Spieler. Auf Stufe F macht er kaum Fehler.

Der Würfel `RND` legt seine drei Ziffern immer in den Registern D, E und F ab. Dort stehen im Programm deshalb nur Hilfswerte, die gerade nicht gebraucht werden. Der Zustand des Spiels liegt in anderen Registern.

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
