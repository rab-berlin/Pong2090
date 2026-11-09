# Pong2090

Pong für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 2 Tastern: der Spieler links gegen den Computer rechts in 15 Spielstufen, oder allein gegen die Wand.

- 256 Programmschritte, alle belegt
- Datei: [pong.mic](pong.mic) (Version 01)

## Spiel

- **Spielstufe:** die erste Hex-Taste nach RUN. 1-F spielt gegen den Computer, je höher, desto stärker. 0 ist das Spiel allein gegen die Wand.
- **Schläger:** je 3 LEDs hoch, der eigene in Spalte 1, der des Computers in Spalte 8.
- **Winkel:** Ein Treffer mit der Schlägermitte spielt flach zurück. Mit der oberen oder unteren Kante geht der Ball steil zurück, weg von der Mitte. Oben und unten prallt der Ball ab.
- **Punkt:** Die Anzeige zeigt Spieler und Computer. Mit einer Hex-Taste geht es weiter.
- **Sieg:** Wer zuerst 9 Punkte hat, gewinnt.
- **Wand (Stufe 0):** Der erste Fehler beendet das Spiel. Die Anzeige zeigt die Treffer hexadezimal.
- **Tempo:** Ein Takt dauert im Mittel 0,95 s.

## Bedienung

| Taster | Pfeiltaste im Emulator | Wirkung |
|---|---|---|
| Eingang 2 | hoch | Schläger hoch |
| Eingang 3 | runter | Schläger runter |

Den Taster halten, bis der Schläger zieht.

## Start

RESET, Matrix einschalten, `HALT` `NEXT` `00` `RUN`, dann die Spielstufe eingeben. Neues Spiel: RESET, RUN.

## Matrixprotokoll

Das Programm verwendet das **frühere Blockprotokoll** der Matrix: Bereit-Signal auf Eingang 1, Startnibble 1, 16 Werte, Endnibble 0. Die Matrix bleibt die ganze Zeit eingeschaltet.

Im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` auf `alt` stellen. Für das aktuelle Protokoll des [ESP2090 Studios](https://github.com/rab-berlin/ESP2090) muss das Programm noch umgestellt werden.
