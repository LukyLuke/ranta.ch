+++
date = "2017-12-25T13:13:13+01:00"
title = "ESP 32 Toolkit Installation"
Categories = ["Development","Tutorial","ESP-32"]
Tags = ["Development","esp-32","esp-wroom-32","kdevelope"]
Description = "Initiale Installation des ESP-32 Toolkits"

+++

Die ESP-32 Mikrokontroller Familie sind ein optimales Produkt für IoT Sachen. Hergestellt werden sie von Espressif Systems und sind ein sogenanntes Low-Power System on a Chip (SoC) mit integriertem WiFi und dual-mode Bluetooth funktionalität. Mehr dazu auf der Webseite [esp32.net](http://esp32.net/) und den Produkseiten von [Espressif](http://espressif.com/en/products/hardware).

Ein Beispiel wie man diese Dinger mit dem Arduino-Studio (oder wie man dem sagen soll) verwenden kann, habe ich schon unter [ESP-32/ESP-WROOM-32 verkabeln und programmieren](/tutorials/ESP-32-Initial/) kurz erleutert. Die Verkabelung und alles um die ROMs hoch zu laden werden auch hier beschrieben.

Um das ganze Potential aus dem Chip heraus zu holen, lohnt es sich aber die schon vorhandenen Libraries und Toolkits von Espressif zu verwenden: [ESP-IDF Programming Guide](http://esp-idf.readthedocs.io/en/latest/get-started/)

# Vorbereitungen

Man braucht nur folgende Packages: `git, gcc, ncurses, flex, bison, gperf, PySerial`
Je nach Distribution sind diese unter andren Namen und os weiter verfügbar.

# Toolchain Setup

Die Toolchain ist relativ schnell installiert und konfiguriert:

```bash
$ mkdir -p ~/Documents/esp/
$ cd ~/Documents/esp/
$ curl -s https://dl.espressif.com/dl/xtensa-esp32-elf-linux64-1.22.0-75-gbaf03c2-5.2.0.tar.gz | tar xzf -
```

# Development Environment

Gleiches gilt für das ganze Development-Environment:
```bash
$ cd ~/Documents/esp/
$ git clone --recursive https://github.com/espressif/esp-idf.git
```

# Umgebungsvariabeln

Nun müssen noch zwei Umgebungsvariabeln gesetzt werden:

```bash
$ echo "export PATH=\$PATH:\$HOME/Documents/esp/xtensa-esp32-elf/bin" >> ~/.profile
$ echo "export IDF_PATH=\$HOME/Documents/esp/esp-idf" >> ~/.profile
```

# Beispiel-Projekte

Einige Beispiele findetman im Verzeichniss `~/Documents/esp/esp-idf/examples`. Von dort kann man sich z.B. das `~/Documents/esp/esp-idf/examples/get-started/hello_world` Projekt kopieren und anpassen:

Die Umgebungsvariabeln müssen ab hier bekannt sein - also neue Konsole, resp. Ab-/Anmelden oder einfach schnell in der Konsole ausführen.

```bash
$ cd ~/Documents/esp/projects
$ cp -r $IDF_PATH/examples/get-started/hello_world .
```

Nun kann das ganze konfiguriert werden:
```bash
$ cd ~/Documents/esp/projects/hello_world
$ make menuconfig
```
Hier muss im besoderen der Serielle Port definiert werden wo der ESP32 angeschlossen ist: `Serial flasher config > Default Serial Port`
Weiter können auch noch Featuers und Funktionennan/ausgeschalten werden, etc.

Über das Kommando `make flash` kann dann das ROM geschrieben werden (Verkabelung beachten zum Flashen!) und mittels `make monitor` die Serielle Schnittstelle gelesen werden. Beim hier kopierten *Hello World* müsste also ein *Hello world!* erscheinen und danach ein Countdown für einen Reboot. Der Monitor-Modus muss mit `CTRL + ]` beendet werden.

Wenn nur binäres Gewirr zu sehen ist, dann muss in der Konfiguration die `ESP32_XTAL_FREQ_SEL = 26MHz` eingestellt werden und neu geflasht werden.

Über `make help` findet man noch mehr Befehle :o)
