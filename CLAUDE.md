# TD Maze Planer

Helfer-Tool zum Planen von Mazes in Tower-Defense-Spielen. **Eine einzige, eigenständige HTML-Datei** (`index.html`, Vanilla JS, Canvas, kein Build, keine Abhängigkeiten, läuft offline). Bedienung und Texte in **Deutsch und Englisch** (Umschalter DE/EN in der Kopfzeile).

## Dateien im Ordner
- `index.html` – das ganze Tool (CSS, HTML, JS in einer Datei).
- `default-map.js` – optionale Startkarte, wird beim Start geladen (siehe unten). Vom Nutzer erzeugt, nicht ändern oder löschen.
- `start-maps.js` – optionale Auswahlliste „Startkarten“ (`window.TD_START_MAPS = [Karte, …]`, siehe unten). Wird vom Button „Zu Startkarten hinzufügen“ erzeugt.
- Gespeicherte Karten/Mazes des Nutzers (`*.tdmap.json`, `*.tdmaze.json`) – nicht anfassen.
- `default-map.jsyxyx` – nicht von uns (vermutlich Sicherungskopie), ignorieren.

## Zwei Modi
1. **Map zeichnen** – Karte anlegen: Raster (Größe änderbar, wählbar an welchen Seiten), Feldtypen (Bebaubar, Blockiert, Fester Weg, Start, Ziel, Checkpoint), Hilfslinien, Spiegeln. Zeichenform ist fest: **Fläche** (Rechteck).
2. **Maze zeichnen** – auf der Karte Türme (1×1, 2×2, 4×4) setzen; der kürzeste Weg wird live berechnet, Statistik, Gegner-Simulation, PNG-Export. Zeichenform ist fest: **Linie**. Das Tool **startet immer im Maze-Reiter**.

## Testen
Der Browser-Pane von Claude Code kann `file://` nicht öffnen. Zum Testen einen kleinen Server starten und über `preview_start` mit `url` öffnen:

```bash
python -m http.server 8766 --directory D:\ClaudeCode\MazePlanner
```
Danach `http://localhost:8766/index.html`. Nach dem Test den Server wieder beenden. Im Pane ggf. `resize_window` (z. B. 1280×800) setzen, sonst hat die Seite Größe 0. Zustand lässt sich per `javascript_tool` prüfen (alle Top-Level-`let/const` wie `map`, `walls`, `route`, `settings` sind erreichbar). Keine Testdateien im Projektordner hinterlassen (v. a. keine `default-map.js` überschreiben).

## Aufbau der Datei
- **CSS** oben; Sichtbarkeit je Modus über `body[data-mode=map|maze] .only-map/.only-maze`.
- **HTML**: Kopfzeile (Modus-Tabs, DE/EN, Undo/Redo, Einpassen), linke Leiste `aside` mit einklappbaren Gruppen (`<section>` mit `<h3>`), `#stage` mit `<canvas id="cv">`, Banner, Toast, Statusleiste.
- **JS** (ein `<script>`, `'use strict'`), grob in dieser Reihenfolge: Sprache (`EN`, `t()`, `applyStaticI18n`) → Zustand (`settings`, `map`, `walls`, …) → Marker → Wegfindung (`shortest`, `los`, `anyLeg`, `computeRoute`, `recompute`) → Zeichenoperationen/Türme → Undo/Redo → Ansicht (Zoom/Pan) → Rendering (`renderWorld`, `draw`) → Eingabe (Pointer/Tastatur) → UI-Sync → Karte ändern (Resize/Spiegeln) → Datei-Laden/Speichern → Verdrahtung → Start.

## Datenmodell
- `map = { name, w, h, cells: Uint8Array (0 bebaubar, 1 blockiert, 2 fester Weg), starts[], goals[], cps[] }`, Punkte als `{x,y}` (Zellkoordinaten). Mehrere Starts und Ziele erlaubt.
- `walls`: `Uint32Array w*h`, `0` = frei, sonst **Turm-ID**. `towerSize: Map(id → 1|2|4)`. Türme werden ganz oder gar nicht gesetzt, Entfernen löscht den ganzen Turm (alle Zellen mit gleicher ID).
- `guides[]`: Hilfslinien `{x1,y1,x2,y2}` in Feldeinheiten, Vielfache von 0,5 (Fangraster = doppeltes Raster), Winkel 0°/45°/90°. Gehören zur **Karte** (im Map-Modus zeichnen, im Maze-Modus nur sichtbar).
- Undo/Redo: `snapshot()`/`restore()` (Zellen, Türme, Marker, Hilfslinien). Verlauf wird beim Moduswechsel und Laden geleert.

## Wegfindung (wichtig)
- Bewegungsmodus `settings.move`: `'any'` (Standard, beliebiger Winkel), `'eight'` (8 Richtungen), `'four'`.
- `'four'`/`'eight'`: Dijkstra auf dem Raster (`shortest`), keine Ecken schneiden.
- `'any'`: exakter kürzester Weg über Sichtbarkeitsgraph an konvexen Hindernis-Ecken + A* (`anyLeg`, `los`/`losR`, `findCorners`). Gegner sind Scheiben: Abstand Mitte–Wand `curR = min(0,5, Breite/2 + Wandabstand)`; Ecken werden als Polygon (2 Punkte je Viertelkreis) abgerundet. Lücken schmaler als die Gegnerbreite sind gesperrt. Schachbrett-Ecken (zwei Hindernisse nur diagonal berührend) sind nie passierbar. Slider-Grenzen: Breite 0,3–1,0, Wandabstand 0,02–0,25 (auch beim Laden geklemmt).
- Route: Start → alle Checkpoints in Reihenfolge → Ziel. Es wird **nur der eine kürzeste Weg über alle Start/Ziel-Kombinationen** gezeichnet (`route.routes[0]`).
- **Wegblock-Schutz** (`settings.prevent`): Jeder Start muss irgendein Ziel erreichen können. Beim Setzen eines Turms wird nur neu geprüft, wenn er das `pathMask` (alle Felder, die einen Startweg beeinflussen) berührt. Schon vorher blockierte Starts zählen nicht (`route.unreachable`).
- Der Weg ohne Türme (`baseRoute`) wird über `baseCache` zwischengespeichert.

## Dateiformate (JSON)
- **Karte** `*.tdmap.json`: `{ format:'td-map', version:1, name, width, height, cells:[Zeilen als Ziffernstrings], starts, goals, checkpoints, guides:[[x1,y1,x2,y2]] }`. Altes Format mit `start`/`goal` (einzeln) und `image` wird weiter gelesen (Bild ignoriert).
- **Maze** `*.tdmaze.json`: `{ format:'td-maze', version:2, name, map:{…komplette Karte…}, walls:[Zeilen '0'/'1'], towers:[[x,y,n]], settings:{ move, diag, enemyW, margin, tower } }`. Die Karte ist **eingebettet**.
- **Startkarte** `default-map.js`: `window.TD_DEFAULT_MAP = {…Karten-JSON…};` Als `.js`, weil Browser bei `file://` kein `fetch` auf Nachbardateien erlauben, `<script src>` aber schon. Wird beim Start per `<script src="default-map.js">` geladen und im Maze-Reiter geöffnet. Erzeugt über „Als Standardkarte speichern“ im Map-Modus.
- **Startkarten-Liste** `start-maps.js`: `window.TD_START_MAPS = [Karten-JSON, …];` Erscheint als Gruppe „Startkarten“ (Auswahl + „Gewählte Karte laden“, lädt nur die Karte, bleibt im aktuellen Modus). `startMaps()` = Liste plus `default-map.js` (gleicher Name: `start-maps.js` gewinnt). „Zu Startkarten hinzufügen“ schreibt die komplette Datei neu (bestehende Karten + aktuelle, gleicher Name wird ersetzt); der Nutzer ersetzt die Datei im Projektordner. Beim Start wird `default-map.js` geladen, sonst die erste Karte der Liste. Fehlt die Datei, bleibt die Gruppe ausgeblendet.

## Datei-Dialoge
Chrome/Edge: File System Access API. Der Browser kennt den Ordner der Seite nicht, daher wählt der Nutzer einmalig die HTML-Datei aus; der Verweis liegt in IndexedDB und dient als `startIn` für alle Dialoge. Andere Browser: normaler Download bzw. `<input type=file>`.

## Mehrsprachigkeit (Konvention, unbedingt einhalten)
- Deutsch ist die Quellsprache. **Statisches HTML:** deutscher Text im Element, englische Fassung im Attribut `data-en` (bzw. `data-en-title`, `data-en-placeholder`). `data-en` ersetzt das gesamte `innerHTML` – nur auf Blattelemente setzen, **nie** auf Container mit Buttons/Inputs, JS-Handlern oder JS-befüllten Kindern (Text dort in ein eigenes `<span data-en>` packen).
- **Dynamische Texte:** immer über `t('deutscher Text {0}', arg)`; der deutsche Text ist der Schlüssel im Objekt `EN`, `{0}`/`{1}` sind Platzhalter. `toast(key, ...args)` übersetzt selbst. Neue Texte in `EN` eintragen.
- Dezimaltrenner über `dsep()`. Nach `setLang()` werden alle dynamischen Teile neu aufgebaut (`buildPalette`, `syncPalette`, `syncMove`, `syncInputs`, `recompute`, …). Sprache liegt in `localStorage` (`tdplaner.lang`).
- Achtung: keine lokalen Variablen namens `t` in Funktionen, die `t()` aufrufen.

## Weitere Konventionen und Stolperstellen
- Speicherstände im Browser (`localStorage`, immer in try/catch): Sprache, eingeklappte Gruppen (`tdplaner.sec.<deutscher Titel>`). Die Gruppen-Ein-/Ausklapper hängen an `h3`-Text; Umbenennen einer Gruppe verwirft ihren gemerkten Zustand.
- Zoom: kleinster Zoom = ganze Karte sichtbar (`fitScale`), Karte bleibt beim Verschieben im Bild (`constrainView`).
- Karte spiegeln (horizontal/vertikal) und Größe ändern (Seiten wählbar, `settings.resizeX/Y`) verschieben Marker, Türme und Hilfslinien mit.
- Vorlage-Bild, Raster-Schalter (G), Zeichenformen Pinsel/Rahmen/Füllen und die untere Statistik-Leiste wurden bewusst entfernt – nicht wieder einbauen, ohne dass der Nutzer es verlangt.
- Nutzer schreibt Deutsch; Antworten und UI-Texte auf Deutsch, Code-Kommentare bisher ebenfalls deutsch.
