+++
Tags = ["Tutorial","MATLAB","Rechnen","Matrizen","Coding"]
Description = "MATLAB Part 3, Plotten von Werten und Funktionen"
date = "2017-02-19T12:00:00+01:00"
title = "MATLAB Part 3, Plotten von Werten und Funktionen"
Categories = ["Tutorial","MATLAB","Lernen","Programmieren"]

+++

# MATLAB - Plotten

Matlab arbeitet eigentlich nur mit Matrizen und nciht mit Zahlen
Ein `>> x = 3;` ist eine `1x1 Matrix` mit dem Wert `3`.

## Variablen

Kein Keyword notwendig.
Wichtig bei Schlaufen etc.: Es sollten die Zeichen `i` und `j` nicht verwendet werden, das sind die Imaginären Zahlen welche dadurch überschrieben werden (`>> im = 5 + 2*i`)

# Plotten von Werten

In MATLAB wird die Funktion `plot(X, Y[], Z])` verwendet um einen Vektor grafisch dar zu stellen.
Die Übergebenen Vektoren müssen logischerweise die geliche Dimension aufweisen so dass jeweils ein Punkt von X mit einem Punkt von Y (und Z) aufgezeichnet werden kann.

## Beispiele

### Einfache 2-Dimensionale Linie

Zuerst werden die zwei Vektoren für die X und die Y Achse definiert und anschliessend der `plot(x, y, z)` Funktion übergeben. Als Resultat wird ein Fenster geöffnet welches eine Linie in einem Karthesischen Koordinatensystem zeigt, welche jeweils den Wert aus dem X-Vektor mit dem Wert aus dem Y-Vektor verbindet.

```matlab
>> x = [ 1, 2, 3, 5, 8 ];
>> y = [ 2, 3, 0, 5, 1 ];
>> plot(x, y);
```

### Einfache 3-Dimensionale Linie

Gleiches Prinzip wie bei der 2-Dimensionalen Linie.

```matlab
>> x = [ 1, 2, 3, 5, 8 ];
>> y = [ 2, 3, 0, 5, 1 ];
>> z = [ 2, 1, 2, 1, 3 ];
>> plot(x, y, z);
```

### Sinus-Funktion

Für jeden Wert auf der X-Achse wird ein Wert in der Y-Achse gebraucht. Zuerst wird also ein Vektor mit den Werten der X-Achse erstellt, daraus dann die Werte von Y mit der Sinus-Funktion. Die Werte auf der X-Achse gehen von `-10` bis `+10` mit einer Distanz von `0.5`, also `101` Punkten.

```matlab
>> x = -10:0.5:10;
>> y = sin(x);
>> plot(x, y);
```

Je feiner man die X-Matrix auflöst, desto schöner wird dann auch die geplottete Funktion.

