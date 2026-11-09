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

## Türsensor: SONOFF SNZB-04P über ZBDongle-P

So kommt das Signal an:

```
SNZB-04P  →  Zigbee  →  ZBDongle-P am Mac  →  Zigbee2MQTT  →  Mosquitto  →  Spieluhr im Browser
```

Die Spieluhr verbindet sich selbst per WebSocket mit Mosquitto (eingebauter MQTT-Client, keine externe Bibliothek). Sie startet, wenn der Sensor von „zu“ auf „offen“ wechselt. Wiederholte Meldungen, Batterieberichte und der Neustart von Zigbee2MQTT lösen nichts aus.

Der Zigbee-Stick braucht Software auf dem Mac, das geht nicht rein im Browser. Docker scheidet auf dem Mac aus, weil Docker Desktop keine USB-Sticks durchreicht. Also nativ installieren:

### 1. Mosquitto (MQTT-Broker)

```
brew install mosquitto
```

In `$(brew --prefix)/etc/mosquitto/mosquitto.conf` am Ende ergänzen:

```
listener 1883 127.0.0.1
listener 9001 127.0.0.1
protocol websockets
allow_anonymous true
```

Beide Listener hören nur auf dem Mac selbst, von außen ist nichts erreichbar. Dann starten (startet ab jetzt automatisch mit):

```
brew services start mosquitto
```

### 2. Zigbee2MQTT

Node.js 22 installieren und Zigbee2MQTT holen:

```
brew install node@22
echo 'export PATH="$(brew --prefix node@22)/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
corepack enable
git clone --depth 1 https://github.com/Koenkk/zigbee2mqtt.git ~/zigbee2mqtt
cd ~/zigbee2mqtt
pnpm install --frozen-lockfile
```

Den Stick einstecken und seinen Anschluss suchen: `ls /dev/cu.*` (Eintrag mit `usbserial` oder `SLAB` im Namen).

Datei `~/zigbee2mqtt/data/configuration.yaml` anlegen:

```yaml
mqtt:
  server: mqtt://127.0.0.1:1883
serial:
  port: /dev/cu.usbserial-XXXX   # dein Anschluss von oben
  adapter: zstack                # ZBDongle-P
frontend:
  enabled: true
  port: 8080
advanced:
  network_key: GENERATE
  pan_id: GENERATE
```

Starten mit `pnpm start`, dann im Browser `http://127.0.0.1:8080` öffnen.

### 3. Sensor koppeln

In der Zigbee2MQTT-Oberfläche Anlernen erlauben („Permit join“), dann am SNZB-04P die Taste etwa 5 Sekunden halten, bis die LED blinkt. Das Gerät danach am besten in `Tuer` umbenennen.

### 4. Spieluhr verbinden

In der Spieluhr unter **Einstellungen → Türsensor** auf **Sensor erkennen** klicken und die Tür einmal öffnen. Fertig. Die Statusanzeige zeigt danach Verbindung, Türzustand, letztes Signal und Batteriestand. Bricht die Verbindung ab oder wird die Batterie schwach, steht das unten links in der Statuszeile.

Chrome fragt beim ersten Verbinden womöglich, ob die Seite auf Geräte im lokalen Netzwerk zugreifen darf. Einmal erlauben, am besten bevor du den Kiosk-Modus startest. Safari blockiert die Verbindung von einer https-Seite zu `ws://127.0.0.1`, deshalb Chrome verwenden.

### 5. Zigbee2MQTT automatisch starten

```
npm install -g pm2
pm2 start "pnpm start" --name zigbee2mqtt --cwd ~/zigbee2mqtt
pm2 save
pm2 startup
```

`pm2 startup` gibt einen Befehl aus, den du einmal mit `sudo` ausführst.

### Zum Testen ohne Sensor

Taste `F9` (änderbar), `window.dispatchEvent(new Event('door-open'))` oder `window.doorOpen()`.

## Dauerbetrieb auf dem Mac mini

Damit nach einem Neustart niemand klicken muss, Chrome vollständig beenden und im Kiosk-Modus mit freigegebenem Ton starten:

```
open -a "Google Chrome" --args --kiosk --autoplay-policy=no-user-gesture-required https://DEIN-NAME.github.io/DEIN-REPO/
```

Erkennt die Seite, dass Ton erlaubt ist, überspringt sie den Startknopf automatisch. Für Bildschirme, die nicht gestört werden sollen: Energiesparen und Bildschirmschoner auf dem Mac deaktivieren.
