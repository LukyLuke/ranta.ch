+++
Tags = ["Tutorial","OpenSSL","Security","Encryption","Decryption","Certificates"]
Description = "OpenSSL Tutorial for Beginners"
date = "2020-08-28T10:00:00+01:00"
title = "Basics: OpenSSL Tutorial"
Categories = ["Tutorial","Security","Kryptografie"]

+++

# Grundlagen zu OpenSSL

Hier werden die einfachsten Befehle und Funktionen von OpenSSL beschrieben um einen kleinen Überblick zum Thema Verschlüsselung, Signierung, Zertifikate, ... zu erhalten.

## Einleitendes zum Thema

Was mit Kryptografie behandelt werden soll sind:

* **Integrität** - Nachricht kommt nachweislich unverfälscht an.
* **Vertraulichkeit** - Nachricht kann nur von der Person gelesen werden für welche sie bestimmt ist.
* **Authentizität** - Der Absender ist eindeutig identifizierbar.
* **Verbindlichkeit** - Der Absender kann nicht verneinen die Nachricht versendet zu haben.

Leider kann man nicht alles mit einer einzigen Methode sicherstellen.

* **Integrität** - Hashes oder Signaturen
* **Vertraulichkeit** - Symetrische und Asymetrische Verschlüsselung
* **Authentizität** - Signatur
* **Verbindlichkeit** - Signatur

## OpenSSL Anwenden

Wir brauchen mindestens eine kleine und eine grössere Textdatei:

```bash
$ echo "Welcome to a very simple OpenSSL-Tutorial LAB" > plaintext.txt
$ echo for X in $(seq 9999); do echo "Welcome to a very simple OpenSSL-Tutorial LAB $X" >> plaintext_long.txt; done
```


### Encode/Decode Base64

Mit `base64` kann alles, egal ob binär oder nicht, nach ASCII codiert und wieder decodiert werden. Dabei werden immer Blöcke von 6 Bits in die Zeichen `a-zA-Z0-9+/` gewandelt.
Das hat den Vorteil, dass man jegliche Datei über egal welchen Kanal versenden kann da die verwendeten Zeichen in allen Charsets verfügbar sind.

```bash
$ echo "Hello OpenSSL!" | openssl enc -base64
$ echo "SGVsbG8gT3BlblNTTCEK" | openssl enc -base64 -d
```


### Verschlüsseln

**Sicherstellen:** Vertraulichkeit / Confidentiallity

Um etwas zu verschlüsseln, wird das `enc` tool verwendet mit einem Algorithums. Für das anschliessende Entschlüsseln muss zusätzlich der Parameter `-d` angegeben werden.

```bash
$ echo "Hello OpenSSL!" | openssl enc -aes-256-cbc -pbkdf2
$ echo "U2FsdGVkX1/1soBuZdmhjwsPRSbdXNV0PCyBz56uldM=" | openssl enc -aes-256-cbc -a -d -pbkdf2
```
_Hier wurde das Password `pass` verwendet._


#### Passwort in einer Datei

Die Passwort-Datei sollte immer nur von dem Besitzer les/schreibbar sein: `chmod 0600 password.txt`. Ansonsten können andere Benutzer auf dem selben Gerät ebenfalls damit arbeiten.

```bash
$ echo "pass" > password.txt && chmod 0600 password.txt
$ echo "Hello OpenSSL!" | openssl enc -aes-256-cbc -pbkdf2 -kfile password.txt
$ echo "U2FsdGVkX1/1soBuZdmhjwsPRSbdXNV0PCyBz56uldM=" | openssl enc -aes-256-cbc -a -d -pbkdf2 -kfile password.txt
```

#### Passwort generieren lassen

OpenSSL kann verwendet werden um Passwörter zu generieren, welche dann als Einmalpasswort verwendet werden können (siehe weiter unten).

```
$ openssl rand -hex -out password.bin 64
```
_Es sollte der Parameter `-hex` verwendet werden und nicht `-base64`._



### Dateien Ver-/Entschlüsseln

Gleiches Vorgehen wie beim beim Verschlüsseln von Texten, nur wird bei Dateien der Parameter `-in FILE` verwendet.

```bash
$ openssl enc -aes-256-cbc -pbkdf2 -in plaintext.txt -out cypher.dat
$ openssl enc -aes-256-cbc -pbkdf2 -in plaintext_long.txt -out cypher_long.dat
```

Und das Entschlüsseln entsprechend:
```bash
$ openssl enc -aes-256-cbc -pbkdf2 -d -in cypher.dat -out decrypted.txt
$ openssl enc -aes-256-cbc -pbkdf2 -d -in cypher_long.dat -out decrypted_long.txt
```


### Asymmeteische Verschlüsselung

**Sicherstellen:** Vertraulichkeit / Confidentiallity
**Gebraucht von:** Signierung (Authentizität und Verbindlichkeit)

Für eine Asymetrische Verschlüsseln wird ein privater und ein öffentlicher Schlüssel gebraucht. Dabei darf der private Schlüssel niemals von einer anderen Partei als dem Besitzer eingesehen oder verwendet werden. Der öffentliche Schlüssel hingegen muss von allen Parteien (und auch unbeteiligten) verwendet und eingesehen werden können.

#### Erstellen der Schlüssel

Für das Schlüsselmanagement wird die option `genrsa` und `rsa` verwendet. Alternativ kann auch `gendsa` und `dsa` verwendet werden wenn ein anderer Algorithmuss verwendet werden soll.

Zuerst wird ein privater Schlüssel ertsellt, anschliessend der öffentliche aus dem privaten heraus generiert/extrahiert.

```bash
$ openssl genrsa -out private.key
$ openssl rsa -pubout -in private.key -out public.pem -outform PEM
```

Details können folgendermassen aus den Schlüsseln herausgelesen werden:
```bash
$ openssl pkey -in private.key -noout -text 
$ openssl pkey -pubin -in public.pem -noout -text 
```


#### Dateien Ver-/Entschlüsseln

Die Schlüssel können nun verwendet werden um kleine Dateien zu ver- und entschlüsseln.

```bash
$ openssl rsautl -encrypt -inkey public.pem -pubin -in plaintext.txt -out cypher.dat
$ openssl rsautl -decrypt -inkey private.key -in cypher.dat -out decrypt.txt
```

Dateien die grösser sind als der öffentliche Schlüssel gehen leider nicht:

```bash
$ openssl rsautl -encrypt -inkey public.pem -pubin -in plaintext_long.txt -out cypher.dat
  139776174908736:error:0406D06E:rsa routines:RSA_padding_add_PKCS1_type_2:data too large for key size:crypto/rsa/rsa_pk1.c:124:
```

Um das zu machen, bracuhen wir eine Kombination aus dem Passwort und der Asymetrischen Verschlüsselung:

```bash
$ openssl rand -hex -out password.bin 64
$ openssl enc -aes-256-cbc -salt -pbkdf2 -in plaintext_long.txt -out cypher_long.enc -kfile password.bin
$ openssl rsautl -encrypt -inkey public.pem -pubin -in password.bin -out password.enc
```
Wir erstellen ein temporäres Passwort, verwenden dieses für die Symmetrische Verschlüsselung und verschlüsseln anschliessend das Passwort asymmetrisch.

Beide verschlüsselten Dateien können dann auf der Gegenseite verwendet werden für das entschlüsseln:

```bash
$ openssl rsautl -decrypt -inkey private.key -in password.enc -out password.dec
$ openssl enc -d -aes-256-cbc -pbkdf2 -in cypher_long.enc -out p.txt -kfile password.dec
```


## Hashes und Signaturen

**Sicherstellen:** Integrität, Authentizität, Vertraulichkeit

**Do not use SHA-1 or MD5 anymore, use SHA-2 (sha256, sha384, sha512, ...)**

### Hash

Hier kann entweder OpenSSL oder ein anderes Tool verwendet werden. Hashes stellen sicher, das eine Nachricht nicht verändert wurde.

```bash
$ sha256sum password.enc
$ openssl dgst -sha256 password.enc
```

Um einen Hash zu verifizieren muss auf der Gegenseite der Hash ebenfalls berechnet werden und anschliessend mit dem übermittelten verglichen werden.
Hashes werden meistens dazu verwendet, Signaturen und öffentliche Schlüssel zu verifizieren. Dies über einen sicheren Kanal wie Telefon oder Mündlich.

### Signatur

Signaturen stellen sicher, das eine Nachricht nicht verändert wurde und das der Absender wirklich der ist, wer er ausgibt zu sein.
Dazu muss ein Schlüsselpar vorhanden sein wie es auch bei der Asymetrischen Verschlüsselung verwendet wird.

Normalerweise wird eine Signatur mit Base64 noch encodiert, so dass man sie über alle verschiedenen Kanäle übermitteln kann.

```bash
$ openssl dgst -sha256 -sign private.key -out plaintext_long.bin.sign plaintext_long.txt
$ openssl enc -base64 -in plaintext_long.bin.sign -out plaintext_long.sign
```

Das signieren geschieht immer mit dem privaten Schlüssel. Damit verifiziert der Absender, dass die nachricht wirklich von Ihmkommt.

Das verifizieren hingegen geschieht mit dem öffentlichen Schlüssel des Absender:

```bash
$ openssl enc -d -base64 -in plaintext_long.sign -out plaintext_long.decrypt.sign
$ openssl dgst -sha256 -verify public.pem -signature plaintext_long.decrypt.sign plaintext_long.text
```


## Zertifikate

Werden verwendet um einen Kommunikations-Kanal ab zu sichern oder auch bei EMail-Verschlüsselungen etc.
Hierbei speziell ist, dass eine dritte Einheit die Identität des Absenders bestätigt, also eine Person/Organisation welcher beide Parteien vertrauen.

### Self-Sign Certificate

Handelt es sich um eine einfach kommunikation, oder man kennt den Absender persönlichund vertraut ihm, kann man auch mit Self-Sign-Zertifikaten arbeiten. Hierbei bestätigt der Ersteller des Zertifikates, das er er ist.

Hat man schon einen Privaten Schlüssel, kann dieser verwendet werden.
```bash
$ openssl req -key private.key -x509 -sha256 -days 365 -new -out cert_certificate.crt
```

Oder man lässt einen neuen erstellen:
```bash
$ openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:4096 -keyout cert_private.key -out cert_certificate.crt
```

Um den Prozess zu vereinfachen kann der Parameter `-subj "/C=CH/ST=Berne/L=Bern/O=ACME Ltd./OU=IT/CN=foobar.local/emailAddress=postmaster@foobar.local"` verwendet werden.


### Signing Request

Braucht man eine dritte Person um die Identität zu bestätigen, muss das Zertifikat ohne Signatur und Zeitangeben erstelt werden. Dies nennt man dann einen CSR - Certificate-signing-Request.

Hat man schon einen Privaten Schlüssel, kann dieser verwendet werden.
```bash
$ openssl req -key private.key -new -out cert_certificate.csr
```

Oder man lässt einen neuen erstellen:
```bash
$ openssl req -nodes -newkey rsa:4096 -keyout cert_private.key -out cert_certificate.csr
```

Auch hier kann der Parameter `-subj ...` verwendet werden.


### Einen CSR signieren

Ist man selber die auserwählte Partei, welche einen CSR signieren soll, also die Identität bestätigt, macht man dies mit dem eigenen privaten Schlüssel und dem CSR von der Partei die man bestätigen soll. Das Resultat ist dann das Zertifikat welches man wieder zurück schickt.

```bash
$ openssl x509 -signkey private.key -in cert_certificate.csr -req -days 365 -out cert_certificate.crt
```


### Zertifikat verifizieren

Als anwender eines Zertifikates sollte man dieses genau verifizieren und anschauen wer der Eigentümer ist und wer es signiert hat.

```bash
$ openssl x509 -in cert_certificate.crt -text -noout
```


### Zertifikat erneuern

Hat man schon einen privaten Schlüssel der nicht kompromitiert ist, kann dieser verwendet werden mit dem alten Zertifikat, um ein neues zu erstellen - Self-Signed oder nicht anhand der oben verwendeten/weggelassenen Parameter.

```bash
$ openssl x509 -in cert_certificate.crt -signkey private.key -x509toreq -out cert_certificate.csr
```

## Server Zertifikate

Braucht man für einen Server den private Key und das Zertifikat, muss man diese entsprechend aus dem PEM oder PFX extrahieren:

```bash
$ openssl pkcs12 -in certificate.pfx -nokeys -out certificate.pem
$ openssl pkcs12 -in certificate.pfx -nocerts -nodes -out private.key
```

Diese kann man dann z.B. bei Nginx entsprechend einbinden:

```
ssl_certificate /etc/nginx/certs/certificate.pem;
ssl_certificate_key /etc/nginx/certs/private.key;
```
