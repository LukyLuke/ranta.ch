+++
Categories = ["Development","Tutorial","ESP-32"]
Tags = ["Development","esp-32","esp-wroom-32","arduino"]
Description = ""
date = "2017-08-17T18:45:59+02:00"
title = "ESP-32/ESP-WROOM-32 verkabeln und programmieren"

+++

Die ESP-32 Mikrokontroller Familie sind ein optimales Produkt für IoT Sachen. Hergestellt werden sie von Espressif Systems und sind ein sogenanntes Low-Power System on a Chip (SoC) mit integriertem WiFi und dual-mode Bluetooth funktionalität. Mehr dazu auf der Webseite [esp32.net](http://esp32.net/) und den Produkseiten von [Espressif](http://espressif.com/en/products/hardware).

Günstige Bezugsquellen findet man bei EBai, Amazon und natürlich Ali-Express.

# Vorbereitung

Man braucht:
* Einen ESP-WROOM-32 oder ESP-32
* Einen USB-to-Serial Adapter
* Eine 3.3V Quelle (wird meistens bei den USB2Serial Adaptern angeboten)

Die einfachste Verkabelung ist, wenn man die +3.3V vom USB2Serial-Adapter mit dem 3V3 und dem EN Pin auf dem ESP-Chip verbindet, sowie den Ground (GND) mit dem Ground GND.
Weiter muss man den TX vom USB2Serial-Adapter mit dem RX auf dem Chip und den RX vom Adapter mit dem TX vom Chip verbinden.
Um eine Firmware auf den Chip zu laden muss man den IO0-Port vom Chip an den Ground (GND) hängen. Der einfachheit halber kann man hier einen Schalter/Taster oder auch einfach ein trennbares Kabel verwenden.

So ist der ESP-32 bereits einsatzbereit:
```
+------------+      +---------------+
| USB2Serial |      |   ESP-x-32    |
|            |      |           3V3 |---+-- +3.3V
|         Tx +------+ Rx         EN |--/
|   GND   Rx +------+ Tx   GND  IO0 |
+----+-------+      +-------+----+--+
     |                      |    |
     +----------------------+----+
                                \ Nur beim Upload
```
**Achtung:** Auf keinen Fall mit +5.0V speisen, die ESP-32 Familie verträgt nur +1.8 - 3.8 Volt!


# Benötigte Libraries für Arduino

Um anständig mit dem ESP-32 zu arbeiten empfiehlt es sich mit dem [Arduino-ESP32 Core](https://github.com/espressif/arduino-esp32) zu arbeiten. Die Installation ist gut erklärt und spielt sich so ab:

* Installation der [Arduino IDE](https://arduino.cc), bei Slackware auch über SlackBuilds installierbar
* PySerial installieren, bei Slackware ist das bei [SlackBuilds](https://slackbuilds.org/repository/14.2/python/pyserial/) verfügbar
** Es kann auch mittels `pip install pyserial` installiert werden
* Neue Hardware installieren: `mkdir -p ~/Arduino/hardware/espressif`
* Das Git-Projekt klonen: `git clone https://github.com/espressif/arduino-esp32.git ~/Arduino/hardware/espressif/esp32`
* Abhängigkeiten installieren: `cd ~/Arduino/hardware/espressif/esp32/tools && python get.py`

## Stack-Trace

Optional kann (empfohlen) der [ESP ExceptionDecoder](https://github.com/me-no-dev/EspExceptionDecoder) installiert werden um einen lesbaren Stacktrace zu erhalten beim debuggen.

# Programm hochladen

Nach einem Neustart der Adruino IDE sind nun die benötigten Hardware Einträge vorhanden. Für einen ESP-WROOM-32 werden folgende Einstellungen benötigt:
* **Board:** FireBeetle-ESP-32
* **Flash Frequency:** 80MHz
* **Upload Speed:** 921600
* **Port:** /dev/ttyUSB0 oder /dev/ttyUSB1 - das kann sich ändern wenn man den Chip resettet

[![Arduino-IDE Konfiguration](/images/tutorials/arduino_esp32/arduino_config.png)](/images/tutorials/arduino_esp32/arduino_config.png "Arduino-IDE Konfiguration")

Um nun das Programm hoch zu laden, muss der `IO0` Pin auf LOW gesetzt werden, also mit dem Ground (GND) verbunden werden. Anschliessend Power-Off und Power-On um den Chip zu resetten. Wichtig ist, dass wenn man einen Taster verwendet um `IO0` auf low zu halten, dieser muss die ganze Zeit gedrückt werden.

Wenn der Chip resettet hat, muss nochmals der Port geprüft werden - dieser kann sich bei diesem Vorgang ändern (Warum auch immer das resetten eine Auswirkung auf meinen USB-to-Serial Adapter hat weiss ich nicht).

Um das Programm hoch zu laden kann man nun einfach die in der IDE zur verwendung gestellte Funktion verwenden (Icons oben links zum kompilieren und hochladen).
Nach dem Hochladen sollte der Port `IO0` wieder vom Ground getrennt werden und der Chip erneut resettet durch Power Off und Power On.


## Einfaches Programm

Eines der einfachsten Programme kann so aussehen:
```
void setup() {
  Serial.begin(115200);
}
 
void loop() {
  Serial.println("Hello from ESP-WROOM-32");
  delay(500);
}
```
Nach dem hochladen kann dann wie gewohnt über den Serial-Monitor in der Arduino-IDE die Meldung angeschaut werden.

# Demos und mehr

Espressif hat im [ESP-IDF Github Repository](https://github.com/espressif/esp-idf) Demos und Beispiele. Diese sind zwar für die esp-idf ausgelegt, zeigen aber wie Bluetooth und WiFi genutzt werden kann.
