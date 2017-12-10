+++
Tags = ["Development","GoLang","Node.js","Composer","Hyperledger","Openledger","BlockChain"]
Description = "Erste Gehversuche mit Hyperledger in einer Virtuellen Umgebung mit Composer"
date = "2017-11-25T16:39:20+01:00"
title = "HyperledgerSetup"
Categories = ["Composer","BlockChain"]

+++

# HyperLedger Setup

HyperLedger wird aktiv entwickelt auf dem [GI https://gerrit.hyperledger.org/r/](https://gerrit.hyperledger.org/r/#/admin/projects/) und ReadOnly Repositories auf dem [Hyperledger GitHub](https://github.com/hyperledger). Die ganzen Sourcen und Go-Sourcen müssen auch im `$GOPATH/src/github.com/hyperledger` Verzeichniss liegen.


## Benötigte Tools

* GoLang wird benötigt, hier in Version 1.9.2 --> Installieren
* Node.js wird in Version 8.9 benötigt, am besten mit `nvm install 8.9.1 ; nvm default 8.9.1` installieren.
* Gulp wird global gebraucht, am einfachsten mittels `npm install --global gulp` installieren

Für die HyperLedger-CA ist eine Linux-Foundation ID notwendig: [Create a Linux Foundation ID](https://identity.linuxfoundation.org/)

## Vorbereitungen

GoLang ist notwendig, also installieren:
```bash
$ cd /opt
$ curl https://storage.googleapis.com/golang/go1.9.2.linux-amd64.tar.gz | tar zxf -
$ export GOROOT=/opt/go
$ export GOPATH=${HOME}/go
$ export PATH=$PATH:~/go/bin
```

Die `$GOROOT` Variable sollte wenn möglich als Systemweite Umgebungsvariable konfiguriert werden, also irgendiwie so was:
```bash
$ echo "export GOROOT=/opt/go" > /etc/profile.d/golang.sh
$ echo "export GOPATH=${HOME}/go" >> /etc/profile.d/golang.sh
$ echo "export PATH=$PATH:~/go/bin" >> /etc/profile.d/golang.sh
$ chmod a+x /etc/profile.d/golang.sh
```

# Composer und HyperLedger

## Die HyperLedger-Fabric

Die [HyperLedger-Fabric](https://gerrit.hyperledger.org/r/#/admin/projects/fabric) bietet eine einfache Möglichkeit, alle Docker-Images und Abhängigkeiten auf einen schlag zu installieren und registrieren. Dies sollte als `root` gemacht werden:

```bash
$ mkdir -p ~/go/src/github.com/hyperledger
$ cd ~/go/src/github.com/hyperledger
$ git clone https://gerrit.hyperledger.org/r/fabric
$ cd fabric
$ make docker-clean
$ make docker

# Sollte schon mit 'docker' gebildet worden sein
$ make peer orderer configtxgen configtxlator cryptogen proto
```

Der Einfachheit halber sollten die erstellten Binaries nun noch in ein Path-Verzeichniss verlinkt werden:
```bash
$ cd mkdir ~go/bin/
$ cd build/bin/
$ ln -s peer orderer configtxgen configtxlator cryptogen ~go/bin/
```


## Die HyperLedger-CA

HyperLedger bringt eine eigene CA mit, die [HyperLedger-Fabric-CA](https://gerrit.hyperledger.org/r/#/admin/projects/fabric-ca).
Diese kann wie die Fabric installiert werden. Dies sollte als `root` gemacht werden:

```bash
$ cd ~/go/src/github.com/hyperledger
$ git clone https://gerrit.hyperledger.org/r/fabric-ca
$ cd fabric-ca
$ make docker-clean
$ make docker

# Sollte schon mit 'docker' gebildet worden sein
$ make fabric-ca-client fabric-ca-server
```

Der Einfachheit halber sollten die erstellten Binaries nun noch in ein Path-Verzeichniss verlinkt werden:
```bash
$ cd mkdir ~go/bin/
$ cd bin/
$ ln -s fabric-ca-client fabric-ca-server ~go/bin/
```


# HyperLedger mit Node.js

Um Node als ChainCode-Sprache zu benutzen braucht es neben Node auch noch NPM und Gulp, sowie die [HyperLedger-Chaincode-Node](https://gerrit.hyperledger.org/r/#/admin/projects/fabric-chaincode-node) Code und die [Fabric-Samples](https://gerrit.hyperledger.org/r/#/admin/projects/fabric-samples).

Die Fabric-Samples und die ChainCode-Node sollten im selben Root-Verzeichniss liegen, dies kann aber als normaler User gemacht werden ausserhalb von `$GOROOT`:
```bash
$ mkdir ~/hyperledger
$ cd ~/hyperledger
$ git clone https://gerrit.hyperledger.org/r/fabric-samples
$ git clone https://gerrit.hyperledger.org/r/fabric-chaincode-node
```

Vorbereitungen im `fabric-chaincode-node` Verzeichniss:
```bash
$ cd ~/hyperledger/fabric-chaincode-node
$ npm install gulp
$ npm install
```

Jetzt muss noch ein einzelner Node hochgefahren werden um Node als ChainCode zu installieren:
```bash
$ gulp channel-init
$ docker ps
CONTAINER ID   IMAGE                               COMMAND                  ...
xxx            hyperledger/fabric-peer:latest      "peer node start"        ...
xxx            hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-..."   ...
xxx            hyperledger/fabric-orderer:latest   "orderer"                ...
xxx            hyperledger/fabric-couchdb:latest   "tini -- /docker-e..."   ...
xxx            hyperledger/fabric-tools:latest     "/bin/bash"              ...
```
Die 5 Docker-Container sind am laufen, alles ist gut für den nächsten Schritt...

Im nächsten Schritt werden die ProtoBufs erstellt und gestartet und zwei Umgebungsvariabeln gesetzt welche von den Tests gebraucht werden:
```bash
$ gulp protos
$ export CORE_PEER_LOCALMSPID=Org1MSP
$ export CORE_PEER_MSPCONFIGPATH=~/hyperledger/fabric-samples/basic-network/crypto-config/peerOrganizations/org1.example.com/users/Admin\@org1.example.com/msp
```

Nun kann man sich auch den einen Node verbinden und Node.js als ChainCode installieren und verwenden:
```bash
$ cd ~/hyperledger/fabric-chaincode-node
$ CORE_CHAINCODE_ID_NAME="mycc:v0" \
  node test/integration/test.js --peer.address grpc://127.0.0.1:7052
```
Wenn das geklapt hat, dann sollte man eine +Registering with peer grpc://127.0.0.1:7052 as chaincode "mycc:v0"+ sehen, sowie zwei weitere welche +Successfully registered+ und +Successfully established+ aussagen. Diese Peer versteht nun also Node.js als ChainCode.

Um es zu testen kann man über die Binaries aus der Fabric sich auf die Peer verbinden und Kommandos absetzen. Dabei sind die Umgebungsvariabeln von oben notnwendig: `CORE_PEER_LOCALMSPID` und `CORE_PEER_MSPCONFIGPATH`

```bash
$ cd ~/go/src/github.com/hyperledger/fabric
$ CORE_LOGGING_PEER=debug \
  ./build/bin/peer chaincode install -l node -n mycc -v v0 -p ~/hyperledger/fabric-chaincode-node/test/integration
$ CORE_LOGGING_PEER=debug \
  ./build/bin/peer chaincode instantiate -o localhost:7050 -C mychannel -l node -n mycc -v v0 -c '{"Args":["init"]}' -P 'OR ("DEFAULT.member")'
```
Damit installiert man den ChainCode +test.js+ (Annahme) aus dem Verzeichniss `~/hyperledger/fabric-chaincode-node/test/integration` und instanziert diesen damit man anschliessend mit der Peer kommunizieren kann.


# Das erste Netzwerk mit Node...

Das geht am einfachsten mit den [HyperLedger Fabric-Samples](https://gerrit.hyperledger.org/r/#/admin/projects/fabric-samples) welche wir ja schon haben:
```bash
$ cd ~/hyperledger/fabric-samples/first-network
$ ./byfn.sh -m generate
$ ./byfn.sh -m up -l node
```
Anstelle von `-l node` kann auch nichts geschrieben werden, dann wird per Default `-l golang` verwendet.

Das starten dauert eine ganze Weile, am Ende sollte eine Nachricht wie diese auf der Konsole sein:
```bash
...
===================== Query on PEER3 on channel 'mychannel' is successful =====================

===================== All GOOD, BYFN execution completed =====================
 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

Um das Netzwerk wieder herunter zu fahren:
```bash
$ ./byfn.sh -m down
```




# Bei Problemen

## Wenn Docker Version 'latest' nicht kennt

Um mit Composer die aktuellste Version von Hyperledger zu verwenden, muss die aktuellste Version mit dem Tag +latest+ versehen werden.
Für die aktuellsten Tags und Version kann im [Docker Repository](https://hub.docker.com/u/hyperledger/) nachgeschaut werden. Zum Zeitpunkt des schreibens war das +x86_64-1.0.4+

Dies sollte aber nicht notwendig sein wenn man alles mit der HyperLedger-Fabric macht...

Hier am Beispiel wie man mit den [HyperLedger Fabric-Samples](https://gerrit.hyperledger.org/r/#/admin/projects/fabric-samples) ein ersted Demo-Netzwerk aufbauen kann:

```bash
$ docker pull hyperledger/fabric-ordered:x86_64-1.0.4
$ docker tag hyperledger/fabric-ordered:x86_64-1.0.4 hyperledger/fabric-ordered:latest

$ docker pull hyperledger/fabric-peer:x86_64-1.0.4
$ docker tag hyperledger/fabric-peer:x86_64-1.0.4 hyperledger/fabric-peer:latest

$ docker pull hyperledger/fabric-tools:x86_64-1.0.4
$ docker tag hyperledger/fabric-tools:x86_64-1.0.4 hyperledger/fabric-tools:latest
```




