# Christmas Door Magic – die Schneekugel-Spieluhr

Eine Schneekugel mit Weihnachtsmann, die bei jeder Türöffnung „aufgezogen“ wird: Die Kugel wackelt, der Schnee wirbelt hoch, der Weihnachtsmann winkt, ein goldener Schlitten fliegt einmal um die Kugel, und auf dem Messingschild steht das Lied, das gerade läuft. Lichterkette und Christbaumbeleuchtung reagieren auf die Musik.

Alles steckt in einer einzigen `index.html`, ohne Build und ohne Installation. Läuft direkt auf GitHub Pages.

## Einrichten

1. `index.html`, `README.md` und den Ordner `music/` ins Repository legen.
2. GitHub Pages für das Repository aktivieren.
3. Seite öffnen und einmal auf **Spieluhr starten** klicken. Der Klick ist nötig, weil Browser Musik sonst blockieren.

## Lieder eintragen

Maus bewegen, unten rechts erscheint **Einstellungen**. Dort lässt sich jedes Lied einzeln anlegen:

- **Titel und Interpret** erscheinen auf dem Messingschild.
- **Audio:** entweder eine Datei hochladen (oder einfach ins Einstellungsfenster ziehen) oder einen Pfad wie `music/a.mp3` bzw. einen Link eintragen.
- **Abspiellänge:** 5 Sekunden bis 10 Minuten, oder „Ganzes Lied abspielen“. Am Ende wird sanft ausgeblendet.
- **Startpunkt im Lied:** damit es direkt beim Refrain losgeht statt beim Intro.
- **Zeitfenster:** optional, z. B. morgens andere Lieder als nachmittags.
- **Lautstärke dieses Lieds:** zum Angleichen unterschiedlich lauter Aufnahmen.
- **Anhören** (▶) spielt das Lied mit genau diesen Einstellungen samt Show ab.

Unter **Ablauf** stellst du Aktivzeit, Reihenfolge (der Reihe nach oder zufällig), die Pause nach einem Lied, Ein- und Ausblenden und die Gesamtlautstärke ein.

**Wichtig:** Hochgeladene Dateien werden im Browser des jeweiligen Geräts gespeichert (IndexedDB), nicht im Repository. Richte die Lieder also direkt auf dem Mac mini ein, auf dem die Spieluhr läuft. Wer die Musik lieber im Repository hat, legt die MP3s in `music/` und trägt den Pfad ein. Die fünf vorbelegten Lieder zeigen schon auf `music/a.mp3` bis `music/e.mp3`.

Bitte nur Musik verwenden, für die eine Lizenz zur öffentlichen Wiedergabe besteht.

## Verhalten

- Die Pause zählt ab dem Ende eines Lieds (Standard 20 Sekunden).
- Geht die Tür während eines Lieds auf, läuft es standardmäßig weiter, und nur der Weihnachtsmann winkt. Alternativ startet ein neues Lied.
- Außerhalb der Aktivzeit schläft der Weihnachtsmann, und es läuft keine Musik.

## Türsensor anschließen

Die Seite reagiert auf drei Arten:

1. **Taste (empfohlen):** Standard ist `F9`, änderbar unter „Auslöser“. Ein Türkontakt, der sich per USB als Tastatur ausgibt (z. B. ein ESP32-S3 mit USB-HID-Firmware), drückt beim Öffnen einfach diese Taste. Damit braucht es keine zusätzliche Bridge-Software.
2. **Ereignis:** `window.dispatchEvent(new Event('door-open'))`
3. **Funktion:** `window.doorOpen()`

## Dauerbetrieb auf dem Mac mini

Damit nach einem Neustart niemand klicken muss, Chrome vollständig beenden und im Kiosk-Modus mit freigegebenem Ton starten:

```
open -a "Google Chrome" --args --kiosk --autoplay-policy=no-user-gesture-required https://DEIN-NAME.github.io/DEIN-REPO/
```

Erkennt die Seite, dass Ton erlaubt ist, überspringt sie den Startknopf automatisch. Für Bildschirme, die nicht gestört werden sollen: Energiesparen und Bildschirmschoner auf dem Mac deaktivieren.
