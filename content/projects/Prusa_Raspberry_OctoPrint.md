+++
Description = ""
date = "2017-03-04T09:00:24+01:00"
title = "Prusa i4 mit einem RaspberryPi und OctoPrint"
Categories = ["Projects","Robotics","3D-Printing"]
Tags = ["Projects","Robotics","RaspberryPi","OctoPrint","Configuration"]

+++

Um meinen 3D-Drucker, ein Prusa i4, sinnvoll zu verwenden, habe ich immer nach einer optimalen, autonomen Lösung gesucht, bei welcher ich einfach meine Modelle hochladen kann und der Rest +passiert+ einfach so von alleine.
Also habe ich angefangen mir auf einem RaspberryPi selber etwas zusammen zu bastelnmit Pronterface und anderen Tools.
So ganz zufrieden war ich aber nie wirklich...

Seit dem Tag an dem ich nach einem Update von Slic3r die Optionen zu [OctoPrint](http://octoprint.org/) gesehen habe, verwende und empfehle ich nur noch dieses Setup.
Hier wie ich mein System aufgebaut habe - ein wenig Linux kenntnisse sind vorausgesetzt, vorallem wenn es um das installieren von paketen und Anhängigkeiten geht (die habe ich nicht mehr alle im Kopf).

![OctoPrint Logo](/images/octoprint/octoprint_logo.png)

# Vorbereitungen

Man braucht:

* ...einen 3D-Drucker
* ...ich empfehle Marlin als Firmware, bei Repetier und Mendelmax hatte ich immer Aussetzer und andere Probleme
* ...einen RaspberryPi
* ...das Kameramodul zum RaspberryPi
* ...oder ein UVC-Kompatible Webcam

## RaspberryPi installieren

Den RaspberryPi normal mit einem simplen Raspbian installieren und betriebsbereit machen. Hierzu gibt es schon genug Anleitungen.

1. Raspbian herunterladen
2. Raspbian auf die Flash entpacken
3. Bildschirm und Tastatur anhängen
4. RaspberryPi starten

Für das ganze Setup wird kein Xorg oder anderer Window-Manager gebraucht, also nur in den Init-3 starten!
Wenn man nicht als `root` arbeitet für den ganzen Setup-Prozess, den verwendeten User den Gruppen `tty, dialout, video` hinzufügen und `sudo` nicht vergessen.


## RaspberryPi Setup

Den RaspberryPi konfigurieren:
```bash
$ raspi-config
```

* Kein X-Server/Window-Manager
* Netzwerk
* SSH-Server
* Webcam - Wenn das Webcam-Modul verwendet wird
* Alles andere nach eigenem Bedarf

### Firmware-Upgrade

Anschliessend empfehle ich mit `rpi-update` alles auf den neusten Stand zu bringen:
```bash
$ apt-get install rpi-update
$ rpi-update
```

Wenn rpi-update nicht mittels apt-get installiert werden kann:
```bash
$ wget https://raw.github.com/Hexxeh/rpi-update/master/rpi-update -O /usr/bin/rpi-update && chmod +x /usr/bin/rpi-update
```

### Pakete installieren

Folgende Pakete müssen installiert werden:
```bash
$ apt-get install python-pip gcc cmake git git-core libav-tools libjpeg-dev
```

### Installation und Setup abschliessen

Nach dem Setup und Update den RaspberryPi neu starten!

Anschliessend sollten alle Pakete aktualisiert werden:
```bash
$ apt-get update
$ apt-get upgrade
```


# Webcam mit MJPG-Streamer

Der Originale [MJPG-Streamer](https://sourceforge.net/projects/mjpg-streamer/) unterstützt das Kamera-Modul vom RaspberryPi nicht, es gibt aber einen Fork von Liam. Jackson unter GitHub: [MJPEG-Streamer mit RaspiCam](https://github.com/jacksonliam/mjpg-streamer).

## Installation

Einfach das Git-Repo klonen und installieren:
```bash
$ cd /opt/
$ git clone https://github.com/jacksonliam/mjpg-streamer.git
$ cd mjpg-streamer/mjpg-streamer-experimental
$ make
```

Wenn alles gut läuft `mjpg-streamer` kompiliert mit einigen input- und output-Plugins. Diese werden direkt in den `mjpg-streamer/mjpg-streamer-experimental` Ordner kopiert und können von dort ausgeführt werden.

```bash
$ cd /opt/mjpg-streamer/mjpg-streamer-experimental
$ ./mjpg_streamer -i "./input_raspicam.so" -o "./output_http.so -p 8080 -w www"
```

Anschliessend sollte man über den Browser direkt darauf zugreifen können: http://RASPI.IP.ADRESSE:8080/

Für eine UVC-Webcam muss noch das Modul geladen werden, der rest bleibt sich gleich:
```bash
$ modprobe uvcvideo
$ ./mjpg_streamer -i "./input_uvc.so" -o "./output_http.so -p 8080 -w www"
```


# OctoPrint installieren

Die installation von [OctoPrint](http://octoprint.org/) braucht relativ viele Python-Module welche einfach mittels `pip` installiert werden können. Das Problem dabei ist, dass bei einer normalen Raspbian-Installation das `/tmp` als `60MB tmpfs` gemountet wird, was im Normalfall total ausreichend ist.

Für pip und die 10k Python-Module ist das jedoch zu wenig, deswegen:
```bash
$ umount /tmp
```


## Herunterladen und Installieren

Auch OctoPrint wieder unter /opt installieren:
```bash
$ cd /opt
$ git clone https://github.com/foosel/OctoPrint.git
```

Bei meiner Installation bin ich bei mindestens einem Python-Modul gescheitert, deswegen sollte mindestens dieses über apt-get installiert werden:
```bash
$ apt-get install python-babel
```

Alle anderen können dann über pip automatisch installiert werden:
```
$ cd /opt/OctoPrint
$ pip install -r requirements.txt
```
Schlägt ein Paket fehl, die Ausgabe prüfen und die Abhängigkeiten auflösen oder das Modul mittels apt-get manuell installieren.


## OctoPrint starten

Sind alle Abhängigkeiten installiert, kann [OctoPrint](http://octoprint.org/) gestartet werden:
```bash
$ cd /opt/OctoPrint
$ ./run serve --iknowwhatimdoing --port 80
```

Das dauert immer eine Weile, anschliessend kann dann über den Webbrowser alles weitere gemacht werden: http://RASPI.IP.ADRESSE/


## Nützliche Einstellungen

Die Settings von [OctoPrint](http://octoprint.org/) werden im Home des jeweiligen Benutzers gespeichert: `~/.octoprint/config.yaml`

### Zugangsbeschränkung

**Unter:** Einstellungen -> Funktionen -> Zugangsbeschränkung

Neuen Benutzer erstellen um den Zugriff zu beschränken


### Webcam

**Unter:** Einstellungen -> Webcam & Zeitraffer

* **Stream URL**     http://RASPI.IP.ADRESSE:8080/?action=stream
* **Snapshot URL**   http://RASPI.IP.ADRESSE:8080/?action=snapshot
* **FFMPEG Pfad**    /usr/bin/avconv


### System-Settings

**Unter:** Einstellungen -> OctoPrint -> Server

* **Restart OctoPrint** /etc/init.d/octoprint restart
* **Restart System**    /sbin/reboot
* **Shutdown System**   /sbin/halt

## Plugins

OctoPrint kann mit vielen [Plugins](http://plugins.octoprint.org/) ausgestattet werden. Sinnvolle und Sinnlose :)

### Telegram

Um ein komplett autonomes System zu haben, welches man auch mal ein paar Stundenunbeaufsichtigt laufen lassen kann, bietet sich das Telegram-Plugin an.

**Unter:** Einstellungen -> PluginManager -> Mehr...
Nach dem Plungin *Telegram* suchen und installieren. Wie man den Telegram-Bot erstellt wird gut auf der GitHub-Seite erklärt: [Create Telegram Bot](https://github.com/fabianonline/OctoPrint-Telegram/blob/stable/README.md#create-telegram-bot).

Wenn der Bot erstellt ist, meine Empfehlung ist dass man die Group und Privacy Dinger abstellt, kann man nach dem eigenen Bot suchen und per Kommando `/start` anfangen mit dem Bot zu sprechen. Jeden benutzer und jede Gruppe in welche man den Bot haben will, muss man dann noch in den OctoPrint Einstellungen freischalten: **Unter:** Einstellungen -> Plugins -> Telegram

### Z-Distance

Dieses Plugin zeigt einem die Z-Distanz in der Sidebar an, an welcher der Printer aktuell ist.

**Unter:** Einstellungen -> PluginManager -> Mehr...
Nach dem Plungin *Z-Distance* suchen und installieren.


## Automatisch starten

### SysV basierend

Folgende Scripts können verwendet werden um MJPG-Streamer und OctoPrint automatisch zu starten und beenden:
```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          octoprint
# Required-Start:    $network $mjpeg-streamer
# Required-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:
# Short-Description: Run Octoprint
### END INIT INFO

PATH=/sbin:/usr/sbin:/bin:/usr/bin

. /lib/init/vars.sh
. /lib/lsb/init-functions

OCTO_DIR=/opt/OctoPrint

do_start() {
        if [ -x $OCTO_DIR/run ]; then
                [ "$VERBOSE" != no ] && log_begin_msg "Starting OctoPrint on port 80"
                cd $OCTO_DIR && ./run serve --iknowwhatimdoing --port 80 &
                [ "$VERBOSE" != no ] && log_end_msg 0
        fi
}

do_stop() {
        if [ -x $OCTO_DIR/run ]; then
                [ "$VERBOSE" != no ] && log_begin_msg "Stopping OctoPrint on port 80"
                ps ax | grep "./run serve --iknowwhatimdoing" | awk '{print $1}' | head -1 | xargs kill -s SIGQUIT
                [ "$VERBOSE" != no ] && log_end_msg 0
        fi
}

case "$1" in
    start)
        do_start
        ;;
    restart)
        do_stop
        sleep 5
        do_start
        ;;
    status|reload|force-reload)
        echo "Error: argument '$1' not supported" >&2
        exit 3
        ;;
    stop)
        do_stop
        ;;
    *)
        echo "Usage: $0 start|stop" >&2
        exit 3
        ;;
esac
```

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          mjpeg-streamer
# Required-Start:    $network
# Required-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:
# Short-Description: MJPEG-Streamer for streaming
### END INIT INFO

PATH=/sbin:/usr/sbin:/bin:/usr/bin

. /lib/init/vars.sh
. /lib/lsb/init-functions

MJPG_DIR=/opt/mjpg-streamer/mjpg-streamer-experimental

do_start() {
        if [ -x $MJPG_DIR/mjpg_streamer ]; then
                [ "$VERBOSE" != no ] && log_begin_msg "Stopping MJPEG-Streaming Server on port 8080"
                $MJPG_DIR/mjpg_streamer -i "$MJPG_DIR/input_raspicam.so -x 1280 -y 720 -fps 5" -o "$MJPG_DIR/output_http.so -p 8080 -w $MJPG_DIR/www" &
                [ "$VERBOSE" != no ] && log_end_msg 0
        fi
}

do_stop() {
        if [ -x $MJPG_DIR/mjpg_streamer ]; then
                [ "$VERBOSE" != no ] && log_begin_msg "Stopping MJPEG-Streaming Server on port 8080"
                ps ax | grep mjpg_streamer | awk '{print $1}' | head -1 | xargs kill -s SIGINT
                [ "$VERBOSE" != no ] && log_end_msg 0
        fi
}

case "$1" in
    start)
        do_start
        ;;
    restart)
        do_stop
        sleep 5
        do_start
        ;;
    status|reload|force-reload)
        echo "Error: argument '$1' not supported" >&2
        exit 3
        ;;
    stop)
        do_stop
        ;;
    *)
        echo "Usage: $0 start|stop" >&2
        exit 3
        ;;
esac
```

Einfach nach `/etc/init.d/octoprint` resp. `/etc/init.d/mjpg-streamer` kopieren und Ausführbar machen: `chmod +x /opt/bin/{octoprint,mjpg-streamer}`
Anschliessend mit den üblichen verdächtigen die Links ins SysV Verzeichniss machen: `update-rc.d octoprint defaults` und `update-rc.d mjpg-streamer defaults`

### SystemD basierend

Dazu müssen im Verzeichniss `/etc/systemd/system/` folgende Dateien erstellt werden:

```bash
# nano /etc/systemd/system/mjpg-streamer.service

[Unit]
Description=MJPG-Streamer

[Service]
WorkingDirectory=/opt/mjpg-streamer/mjpg-streamer-experimental/
ExecStart=/opt/mjpg-streamer/mjpg-streamer-experimental/mjpg_streamer -i "input_raspicam.so -x 1280 -y 720 -fps 5" -o "output_http.so -p 8080 -w www"
KillMode=process

[Install]
WantedBy=default.target
```

```bash
# nano /etc/systemd/system/octoprint.service

[Unit]
After=networking.service
Description=OctoPrint 3D Printer Software

[Service]
WorkingDirectory=/opt/OctoPrint/
ExecStart=/opt/OctoPrint/run serve --iknowwhatimdoing --port 80
KillMode=process

[Install]
WantedBy=default.target
```

Diese müssen dann noch registert werden:
```bash
# systemctl enable mjpg-streamer.service
# systemctl enable octoprint.service
```

Und natürlich gestartet: `service mjpg-streamer start` sowie `service octoprint start`

Um zu prüfen ob das ganze geklappt hat: `journalctl -u mjpg-streamer.service` resp. `journalctl -u octoprint.service`

# Slic3r Integration

Um den ganzen Prozess zu vereinfachen, kann man bei [Slic3r](http://slic3r.org/) ab Version 1.9.2 (oder schon früher) den [OctoPrint](http://octoprint.org/)-Server als Option eingeben und anschliessend wenn man alles plaziert hat, den gCode direkt auf den Server laden. Anschliessend muss man dann nur noch im WebInterface das File auswählen und drucken.

Dazu wechselt man in Slic3r auf den Tab *Printer Settings* und gibt dort bei *OctoPrint upload* den Host und API-Key an.
Den API-Key kann man auf dem Webinterface in den Einstellungen einsehen und neu generieren.

Ist das gemacht, einfach wenn man alle Objekte positionliert hat (im Tab *Plater*) über die Schaltfläche *Send to printer* den gCode hochladen.
