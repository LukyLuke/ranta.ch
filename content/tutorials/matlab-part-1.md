+++
Tags = ["Tutorial","MATLAB","Rechnen","Matrizen","Coding"]
Description = "MATLAB Part 1, Grundlagen"
date = "2017-02-19T10:00:00+01:00"
title = "MATLAB Part 1, Grundlagen"
Categories = ["Tutorial","MATLAB","Lernen","Programmieren"]

+++

# MATLAB - Grundlagen

Matlab arbeitet eigentlich nur mit Matrizen und nicht mit Zahlen
Ein `>> x = 3;` ist eine `1x1 Matrix` mit dem Wert `3`.

Unter [https://ch.mathworks.com/programs/trials/trial_request.html?prodcode=ML](Free Product Trial) kann eine freie Trial-Lizenz angefragt werden.


## Variablen

Kein Keyword notwendig.
Wichtig bei Schlaufen etc.: Es sollten die Zeichen `i` und `j` nicht verwendet werden, das sind die Imaginären Zahlen welche dadurch überschrieben werden (`>> im = 5 + 2*i`)


# Eingabe von Matrizen und Werten

```matlab
>> x = [ 1, 2, 3 ]; // Zeilenvektor 1x3
>> y = [ 1; 2; 3 ]; // Spaltenvektor 3x1
>> m = [ 1,2,3 ; 4,5,6 ; 7,8,9 ]; // Eine 3x3 Matrix
>> m = 1:5; // Ein Zelenvektor von 1 bis 5
>> m = 1:0.5:5; // Ein Zelenvektor von 1 bis 5 mit 0.5er Schritten (siehe linspace(a,b,s))
>> r = rand(3, 4); // Eine 3x4 Matrix mit Random-Werten zwischen 0 bis 1
>> z = zeros(3, 4); // Eine 3x4 Matrix mit 0en
>> o = ones(3, 4); // Eine 3x4 Matrix mit 1en
>> x(1, 4) = 10; // Der Matrix x in Zeile 1 Spalte 4 den Wert 10 zuweisen
```


## Rechnen mit Matrizen

Matrix für Beispiele:
```matlab
A = [ 1, 2, 3, 4, 5 ;
      5, 6, 7, 8, 9 ;
      4, 3, 2, 1, 0 ;
      8, 7, 6, 5, 4 ]
```

* Eine Matrix wird mit dem Hochkomma transformiert: `>> y = x'`
* Auslesen eines Wertes: Hier Beispielshaft der Wert in der zweiten Zeile und zweiten Spalte `>> A(2, 2)`
  Als Resultat kommt dann also `ans = 6`
* Auslesen einer Spalte: Hier Beispielshaft die zweite Spalte `>> A(:, 2)`
  Als Resultat kommt dann also `ans = 2 ; 6 ; 3 ; 7` (ein Spaltenvektor)
* Auslesen einer Zeile: Hier Beispielshaft die dritte Zeile `>> A(3, :)`
  Als Resultat kommt dann also `ans = 4, 3, 2, 1, 0` (ein Zeilenvektor)
* Auslesen von mehreren Werten: `>> A([1, 3], 4)`
  Als Resultat kommt hier nun der 1 und 3 Wert aus der Spalte 4, ein Spaltenvektor: `ans = 4 ; 1`
  Für einen Zeilenvektor entsprechend umgekehrt: `A(3, [1, 4])` was den Vektor `ans = 4, 1` ergibt
  Oder für eine Submatrix: `A([1, 3], [1, 4])` was dann den Wert `ans = [ 1, 4 ; 4, 1 ]` also aus den Zeilen 1 und 3 jeweils die Werte an Position 1 und 4.


**ACHTUNG: MATLAB rechnet numerisch mit Matrizen**
Die normalen arithmetischen Funktionen `*, -, +, ^` werden als Matrix/Skalar-Funktionen angewendet und nicht als arithmentische Funktionen.
Will man mit einer Matrix elementweise rechnen, muss man einen `.` vor den Operator setzen: `.*, .-, .+, .^`.

```matlab
>> x = [ 1, 2, 3, 4, 5 ];  // Ein 1x3 Zeilenvektor
>> y = x';                 // Ein 3x1 Spaltenvektor
>> x * y
ans = 55
```

Hier wird also ein Skalarprodukt gemacht nach folgendem Schema:
```matlab
              y
              
              1
              2
              3
              4
              5
x  1 2 3 4 5 55
```

Hingegen würde `>> y * x` folgendes Skalarprodukt ergeben:
```matlab
x  1   2   3   4   5
1  1   2   3   4   5
2  1   4   6   8  10
3  3   6   9  12  15
4  4   8  12  16  20
5  5  10  15  20  25
y
```

Arithmentische Berechnungen muss man immer Elementweise machen, Ausnahme ist das Skalarprodukt mit einer 1x1 Matrix:
```matlab
>> x = [ 1, 2, 3, 4, 5 ];
>> x * 10
ans = [ 10, 20, 30, 40, 50 ]

>> x .^ 2
ans = [ 1, 4, 9, 16, 25 ]
```

### Beispiele
Gegebenen ist folgende Matrix:
```matlab
A = [ 1, 2, 3, 4, 5 ;
      5, 6, 7, 8, 9 ;
      4, 3, 2, 1, 0 ;
      8, 7, 6, 5, 4 ]
```

Aus dieser sollen nun unterschiedliche neue Matrizen erstellt werden.


#### 1.) Neue Matrix aus jeder zweiten Spalte

Das kann man nun mit mehreren Wegen lösen.
Entweder man gibt die spalten fix an, was unschön ist:
```matlab
A1 = A(:, 2, 4)
ans = [ 2, 4 ;
        6, 8 ;
        3, 1 ;
        7, 5 ]
```

oder verwendet eine Submatrix welche die gewünschten Spalten berechnet:
```matlab
A2 = A(:, 2:2:end )
ans = [ 2, 4 ;
        6, 8 ;
        3, 1 ;
        7, 5 ]
```

Hier verwendet man nun eine berechnete Matrix `2:2:end` welche wie oben gezeigt eine Matrix erstllt die bei `2` beginnt und zweierschritte macht. Das Keyword `end` kann hier verwendet werden anstelle der `5` (also Anstelle der maximalen Anzahl Spalten) - Es referenziert auf die Matrix `A` in elcher wir uns ja befinden.

#### 2.) Neue Matrix aus jeder zweiten Spalte und jeder zweiten Zeile

Hier das selbe Prinzip: Man verwendet eine berechnete Matrix um die neue aus der gegebenen zu erstellen:
```matlab
A3 = A(2:2:end, 2:2:end)
ans = [ 6, 8 ;
        7, 5 ]
```

So kann eine grosse Matrix, Beispielsweise `B = rand(10)`, schnell in eine neue transformiert werden.



# Datentypen

Standardmässig sind alle Zahlen in MATAB `double`.
Um explizip einen anderen Datentypen zu erhalten, muss man die Casting-funktionen verwenden:

* `>> x = uint8(VAR)`: Unsigned Integer mit 8 Bit.
* `>> x = logical(VAR)|false|true`: Logical aka Boolean.
* `>> x = 'text'`: char, also normaler Text.



# Wichtige Befehle

* `clc` Den Workspace leeren.
* `clear VAR|all` Eine einzelne oder alle Variablen löschen.
* `zeros(X, Y, ...)` Eine mit 0 gefüllte Matrix erstellen mit den gegebenen Dimensionen.
* `ones(X, Y, ...)` Eine mit 1 gefüllte Matrix erstellen mit den gegebenen Dimensionen.
* `linspace(FROM, TO, NUM)` Einen Zeilenvektor vom FROM bis TO mit NUM Zahlen (nicht Schritten) erstellen.
* `logspace(FROM, TO, NUM)` Einen Zeilenvektor vom 10^FROM bis 10^TO mit NUM Zahlen (nicht Schritten) erstellen.
* `logspace(log10(FROM), log10(TO), NUM)` Einen Zeilenvektor vom FROM bis TO mit NUM Zahlen (nicht Schritten) erstellen - Äquivalent zu `linspace()`.
* `size(VAR)` Gibt die Grösse der Matrix zurück, also bei einer 3x4 Matrix ein `ans = 3  4`.
* `length(VAR)` Gibt die grössere Grösse der Matrix zurück, also bei einer 3x4 Matrix ein `ans = 4`.
* `rank(VAR)` Bestimmt den Rang der Matrix, welcher im normalfall gleich der Dimensin ist (ausser in speziellen Fällen).
* `det(VAR)` Betsimmt die Determinante der Matrix.
* `plot(X, Y, Z)` Zeigt die eingegebenen Vektoren grafisch, Z ist optional.

