# TD Maze Planner

**Deutsch** · [English](#english)

Ein Werkzeug zum Planen von Mazes in Tower-Defense-Spielen. Karte zeichnen, Türme setzen, und der kürzeste Weg der Gegner wird live berechnet – so lässt sich ein Maze ausprobieren, bevor man es im Spiel baut.

Das ganze Tool ist **eine einzige HTML-Datei** (Vanilla JS + Canvas): kein Build, keine Abhängigkeiten, läuft offline direkt im Browser. Oberfläche auf Deutsch und Englisch.

## Funktionen

**Map zeichnen**
- Raster mit frei wählbarer Größe (Seiten, an denen vergrößert/verkleinert wird, wählbar)
- Feldtypen: Bebaubar, Blockiert, Fester Weg, Start, Ziel, Checkpoint – mehrere Starts und Ziele möglich
- Hilfslinien (0°/45°/90°, Fangraster ½ Feld), Karte horizontal/vertikal spiegeln

**Maze zeichnen**
- Türme in 1×1, 2×2, 2×2 ★ (Zweitfarbe, z. B. für Carry-Türme) und 4×4 setzen
- Kürzester Weg live berechnet: Start → alle Checkpoints → Ziel
- Bewegungsmodi: beliebiger Winkel (exakt, mit Gegnerbreite und Wandabstand), 8 oder 4 Richtungen
- Wegblock-Schutz: Türme, die einen Start vom Ziel abschneiden würden, werden verhindert
- Statistik (Weglänge im Vergleich zur Karte ohne Türme), Gegner-Simulation, PNG-Export
- Undo/Redo, Zoom und Verschieben

**Dateien**
- Karten (`*.tdmap.json`) und Mazes (`*.tdmaze.json`, Karte eingebettet) speichern und laden
- Alternativ als Text kopieren/einfügen – praktisch in der Discord-Aktivität, wo Downloads blockiert sind
- Mitgelieferte Startkarten (`start-maps.js`), z. B. `SM_red`, `SM_blue`, `GemTD_SC2`

## Benutzung

`index.html` im Browser öffnen – fertig. Das Tool startet im Maze-Reiter mit der Standardkarte.

| Taste | Map-Modus | Maze-Modus |
|---|---|---|
| Linksklick/Ziehen | Fläche zeichnen | Türme setzen |
| Rechtsklick | Feld zurücksetzen | Turm entfernen |
| `1`–`6` / `1`–`4` | Feldtyp wählen | Turmgröße (1×1, 2×2, 2×2 ★, 4×4) |
| `H` / `P` | Hilfslinien-Werkzeug | Weg ein-/ausblenden |
| Leertaste + Ziehen | Ansicht verschieben | Ansicht verschieben |
| `Pos1` | Einpassen | Einpassen |
| `Strg+Z` / `Strg+Y` | Rückgängig / Wiederholen | Rückgängig / Wiederholen |

## Dateien

| Datei | Inhalt |
|---|---|
| `index.html` | Das komplette Tool |
| `start-maps.js` | Liste der Startkarten |
| `default-map.js` | Name der Karte, die beim Start geladen wird |
| `privacy.html`, `terms.html` | Datenschutzerklärung und Nutzungsbedingungen |

Du hast eine eigene Karte, die dauerhaft als Startkarte ins Tool soll? Melde dich gern per [Issue](../../issues) mit der Karte (`*.tdmap.json` oder als Text kopiert), dann nehme ich sie in `start-maps.js` auf.

Fragen, Fehler, Wünsche: bitte als [Issue](../../issues) melden.

---

<a id="english"></a>

# TD Maze Planner (English)

A tool for planning mazes in tower defense games. Draw a map, place towers, and the enemies' shortest path is computed live – try out a maze before building it in-game.

The whole tool is **a single HTML file** (vanilla JS + canvas): no build step, no dependencies, works offline right in the browser. UI in German and English.

## Features

- **Map mode:** resizable grid, tile types (buildable, blocked, fixed path, start, goal, checkpoint; multiple starts/goals), guide lines, mirroring
- **Maze mode:** 1×1, 2×2, 2×2 ★ (alternate color) and 4×4 towers; live shortest path through all checkpoints; any-angle (exact, with enemy width and wall margin), 8- or 4-direction movement; path-block protection; stats, enemy simulation, PNG export; undo/redo, zoom and pan
- **Files:** save/load maps (`*.tdmap.json`) and mazes (`*.tdmaze.json`), or copy/paste them as text (handy in the Discord activity); bundled start maps

## Usage

Open `index.html` in a browser. Keyboard shortcuts are listed in the status bar and in the table above.

Have a map you would like added permanently as a start map? Feel free to contact me via an [issue](../../issues) with the map (`*.tdmap.json` or copied as text) and I will add it to `start-maps.js`.

Questions, bugs, ideas: please open an [issue](../../issues).

---
