+++
Categories = ["Development","Tutorial","Robotics","Linux"]
Tags = ["Development","Arduino","KDevelope","Slackware","Linux"]
Description = ""
date = "2017-03-05T10:41:12+01:00"
title = "Adruino programmieren mit KDevelope"

+++

Wie jeder Programmierer hat man seinen eigenen Favoriten was die Entwicklungsumgebung angeht. Das ist bei mir Slackware64-current mit KWrite und KDevelop, sowie Yakuake als Konsole - und definitiv nicht das grauslige Java-Arduino-Studio :)

Einen Allgemeinen Artikel wie man das zusammenbringt findet man auf dem [Arduino Playground](http://playground.arduino.cc/Code/Kdevelop).

# TL;DR;

0. KDevelop installieren
1. Die AVR-Toolchain installieren
2. Die neuste `adruino.zip` herunterladen: [New Template for Kdevelop/CMake for Arduino 1.0](http://forum.arduino.cc/index.php?topic=244741.0)
3. Entpacken und als tar.bz2 neu verpacken: `unzip arduino.zip && cd arduino && tar -cjf ../arduino.tar.bz2 .`
3.1 Für alle nach `/usr/share/[kde4/]apps/kdevappwizard/templates/` kopieren
3.2 Oder als Egomane nach `~/.kde/share/apps/kdevappwizard/templates/` kopieren
4. Have fun...


# Im Detail

## Die komplette AVR-Toolchain installieren

Bei Slackware geht das am einfachsten über [SlackBuilds.org](https://slackbuilds.org/), andere Distributionen haben das schon im Package-Manager enthalten.

* `avr-binutils` Benötigte Tools für das Cross-Compiling
* `avr-gcc` Cross-Compiling für AVR
* `avr-gdb` für das debuggen mittels GDB
* `avr-libc` beinhaltet die C-Library
* `avrdude` für den Up-/Download der Firmware
* Optional `avarice` wenn man mittels JTag debuggen will
* Optional `avra` für Atmel Assembler

## KDevelop Templates

Zuerst braucht man das neuste `arduino.zip` aus dem Thread [New Template for Kdevelop/CMake for Arduino 1.0](http://forum.arduino.cc/index.php?topic=244741.0)

Dieses zip muss man neu packen als `tar.bz2`, also entpacken und neu verpacken: `unzip arduino.zip && cd arduino && tar -cjf ../arduino.tar.bz2 .`

Will man das Template für alle Benutzer zur Verfügung stellen, dann muss man es nach `/usr/share/[kde4/]apps/kdevappwizard/templates/` kopieren.
Will man das Template nur für sich, dann kann man es nach `~/.kde/share/apps/kdevappwizard/templates/` kopieren.

## Neues Projekt

Startet man KDevelop und erstellt ein neues Projekt, hat man unter dem Punkt **C/C++** nun das Template **Arduino** welches man verwenden kann.

![KDevelop New Project Wizard](/images/tutorials/kdevelope_arduino_new_project.png)

Nachdem das Projekt erstellt wurde, hat man ein neues Demo-File und ein paar Libraries welche man gut gebrauchen kann.

![KDevelop Project Structure](/images/tutorials/kdevelope_arduino_new_project_files.png)

Damit die includes gefunden werden, muss noch der avr-include Path konfiguriert werden. Das macht man am einfachsten über die Project-Settings, wo man den Pfad `/usr/avr/include/` hinzufügt:

![KDevelop Project Structure](/images/tutorials/kdevelope_arduino_new_project_settings.png)

Was man nun noch anpassen muss ist die `CMakeLists.txt` mit einer eigenen Konfig unter dem `platform` Ordner (analog zu dem schon bestehenden `arduino_leonardo`) oder man importiert die `libarduino.cmake` und definiert seine eigenen Parameter:
```cmake
set(ARDUINO_BOARD "AVR_LEONARDO")
set(ARDUINO_VARIANTS "leonardo")
set(ARDUINO_MCU "atmega32u4")
set(ARDUINO_FCPU "16000000L")
set(ARDUINO_UPLOAD_PROTOCOL "avr109")
set(ARDUINO_UPLOAD_SPEED "57600")
set(ARDUINO_PORT "/dev/ttyACM0")

include(${CMAKE_SOURCE_DIR}/platforms/libarduino.cmake)
```

### Template anpassen

Das aktuelle Template hat immer den *Arduino Leonardo* als Standard drin. Wenn man das nicht will, resp. mit einem *Arduino Nano* oder anderen Board arbeitet, kann man das Template auch entsprechend anpassen. Also entpacken, Anpassen und neu verpacken (wie oben erleutert).

