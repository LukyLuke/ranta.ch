+++
Tags = ["Tutorial","MATLAB","Rechnen","Matrizen","Coding"]
Description = "MATLAB Part 2, Programmiergrundlagen"
date = "2017-02-19T11:00:00+01:00"
title = "MATLAB Part 2, Programmiergrundlagen"
Categories = ["Tutorial","MATLAB","Lernen","Programmieren"]

+++

# MATLAB - Funktionen und Schlaufen

Matlab arbeitet eigentlich nur mit Matrizen und nciht mit Zahlen
Ein `>> x = 3;` ist eine `1x1 Matrix` mit dem Wert `3`.

## Variablen

Kein Keyword notwendig.
Wichtig bei Schlaufen etc.: Es sollten die Zeichen `i` und `j` nicht verwendet werden, das sind die Imaginären Zahlen welche dadurch überschrieben werden (`>> im = 5 + 2*i`)


# Programmieren in MATLAB

Normalerweise werden Skripte nicht direkt in MATLAB eingegeben, man kann das aber auch machen.
Skripte sind nichts anderes als dass, was man im Command-Prompt eingibt, in einer Datei ausgelagert, so dass man nicht immer alles neu eingeben muss.

## Wo sucht MATLAB nach Skripten

Generell sucht MATLAB im aktuellen Verzeichniss und vielen anderen nach Skripten. Diese findet man heraus mit dem Kommando `>> path`. Weitere Kommandos sind `pwd` welches das aktuelle Verzeichniss anzeigt und `ls` um den Inhalt an zu zeigen respektive `cd DIR` um in eines zu wechseln.

Um ein Verzeichniss zu dem `path` hinzuzufügen, so dass MATLAB da drin nach unseren Skripten sucht, muss dies über die *Current Folder*-View der Ordner gefunden werden und mittels Kontextmenü auf *Add to Path -> Selected Folders (and Subfolders)* Funktion gemacht werden.


## Editor für Skripte

Wenn man sich nicht mit den Emacs-Tastenkombinationen auseinandersetzen will, dann sollte man unter *Preferences -> MATLAB -> Keyboard -> Shortcuts* die aktiven Einstellungen von *Emacs Default Set* auf *Windows Default Set* stellen oder diejenigen Shortcuts anpassen die nicht passen.

Neue Skripte/Funktionen einfach *New -> Skript* erstellen und Coden (Funktionen auch über *New -> Skript* erstellen). Eine praktische Ansicht ist, wenn man das Editor-Window an das normale MATLAB andockt.

Im Editor kann über den **Run**-Button das Skript ausgeführt und getestet werden, oder man gibt im Command-Promt den Namen des Skripts ein.


# Syntax, Funktionen und Schlaufen

* Kommentare werden mit `% COMMENT` definiert.
* Blöcke kann man mit `%% BLOCKNAME` definieren.
  Ein Block kann verwendet werden um diesen Separiert über **Run Section** ausgeführt werden.
  Zusätzlich dient eine Section auch der visuellen separierung.
* Funktionen, Schlaufen und Bedingungen werden nicht mit Klammern sondern in Pascal-ähnlicher Syntax geschrieben: `function ... end`.

## Globale Funktionen
```matlab
function RESULT = NAME(ARG...)
...
end
```

Der Funktionsname `NAME` sollte niemals gleich heissen wie eine schon bestehende Funktion von uns oder MATLAB.

Das Keyword `RESULT` definiert, welche Variable innerhalb der Funktion als Rückgabe verwendet wird. `RESULT` kann auch im Format einer Matrix angegeben werden: `[a, b ; c, d]` um so eine 2-Dimensionale Matrix aus den vier Variabeln zurück zu geben.

Die Parameter `ARGS...` sind optional und werden 1:1 in die Funktion rein gegeben. Diese können dann innerhalb der Funktion verwendet werden.

### Beispiel einer globalen Funktion
```matlab
function y = test(x)
  % Erstellt eine Matrix befüllt mit '1' mit der gleichen Dimension und gibt diese zurück
  y = ones(size(x));
end

```

## Lokale Funktionen

Lokale Funktionen sind alle Funktionen, welche nicht so heissen wie der Dateiname. Diese sind nicht von aussen, also aus dem Command-Prompt ausführbar.

## Wenn-Dann / If-Else

```matlab
if (BEDINGUNG)
...
else if (BEDINGUNG 2)
...
else
...
end
```

Wenn man je nach Situation reagieren will, dann muss man mit einer IF-ELSE Frage erst definieren was denn geprüft werden soll und was beim Zutreffen der Bedingung gemacht werden soll.

### Beispiel einer Abfrage

Ein sinnloses Beispiel, welches aber veranschaulicht wie man `if - else - end` verwendet:

```matlab
>> a = 3;
>> b = 5;
>> if (a < b)
c = a;
else
c = b;
end

>> c
ans = 3
```

Es wird also verglichen, ob `a` kleiner ist als `b` und wenn dies zutrifft, dann wird der neuen Variable `c` der Wert von `a` zugewiesen. Wäre `a` grösser oder gleich gross wie `b`, so würde die Bedingung nicht zutreffen und `c` wäre dann `5`, also das gleiche wie `b`.

## Schlaufe / For

```matlab
for n = MATRIX
...
end
```
***Wichtig:** Niemals die Variabelnnamen `i` und `j` verwenden da dies imaginäre Zahlen sind!*

### Beispiel einer For-Schlaufe

Beispielsweise kann so eine Matrix durchlaufen werden und die Werte mit sich selber multipliziert werden:

```matlab
>> y = 1;
>> for n = [ 2, 4, 6 ]
y = y * n;
end
>> y
ans = 48
```

Anstelle der Matrix kann man auch hier wieder die dynamisch generierten Matrizen verwenden:

```matlab
>> for n = 1:10
x(n) = n * 5;
end
>> x
ans = [ 5, 10, 15, 20, 25, 30, 35, 40, 45, 50 ]
```


# Konmplettes Beispiel

Zuerst definieren wir eine Funtion `test` in der Datei `test.m`:

```matlab
function y = test(x)
  y = zeros( size(x) )

  for n = 1:length(x)
    if x(n) > 4
      y(n) = two_square( x(n) )
    else if x(n) > 0
      y(n) = double( x(n) )
    else
      y(n) = x(n)
    end
  end
end

function y = two_square(x)
  y = 2 .^ x;
end

function y = double(x)
  y = 2 * x;
end
```

Das ganze kann nun in einem Skript verwenden:
```matlab
clear all;

x = -2:0.01:5;
y = test(x);
plot(x,y)
```
Es wird also zuerst der ganze Stack geleert und anschliessend ein x-Vektor initialisert und davon dann der y-Vektor mit der `test(x)` Funktion berechnet. Das ganze dann zur Veranschaulichung noch ausgegeben mit derplot-Funktion.

