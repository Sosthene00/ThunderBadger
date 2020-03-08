[ [Intro](README.md) ] -- [ [Préparatifs](thunderbadger_10_preparations.md) ] -- [ [Thunder Badger](thunderbadger_20_ThunderBadger.md) ] -- [ [Bitcoin](thunderbadger_30_bitcoin.md) ] -- [ [LND](thunderbadger_40_lnd.md) ] -- [ [Mainnet](thunderbadger_50_mainnet.md) ] -- [ **Bonus** ]

-------
### Thunder Badger : un nœud Bitcoin et ⚡Lightning️⚡ dans votre vieux portable pourri !
--------

## Bonus : Eclair

*Difficulté: facile*

### Introduction
# Lightning: Éclair

Dans ce guide, je vous propose par défaut d'utiliser LND comme implémentation de Lightning. 

Toutefois, il existe d'autres implémentations, dont 2 avec un niveau de développement équivalent à LND, [c-lightning](https://github.com/ElementsProject/lightning) et [Éclair](https://github.com/ACINQ/eclair).

Chaque implémentation a ses points forts et ses points faibles, il n'y a pas de "meilleur" choix dans l'absolu. Je pense qu'il est intéressant d'essayer plusieurs implémentations, c'est pourquoi je vous propose ici d'installer Éclair à la place de LND.

**Note: il existe deux versions d'Éclair, avec ou sans interface graphique. Étant donné que nous sommes censé pouvoir tout faire via SSH, je vais utiliser la version sans interface graphique dans ce tuto.**

**ATTENTION: contrairement au tuto principal, nous allons ici directement sur le mainnet. Deux raisons: je présume que si vous êtes arrivé ici, c'est que vous avez déjà fait le tutoriel et que vous êtes déjà sur le mainnet. La seconde, c'est qu'il n'y a personne sur le testnet de Lightning...**

### Installation
Maintenant, les choses sérieuses : télécharger et vérifier Éclair.

* Vous aurez besoin de Java. Le plus simple est d'installer le package disponible dans Ubuntu. Connecté en tant que `admin`, exécutez d'abord :  
`sudo apt-get -y install openjdk-11-jdk`

À présent en tant que votre utilisateur `bitcoin`, téléchargez :
* La dernière release stable d'Éclair (0.3.3 au moment de la dernière révision de ce tuto, vous pouvez vérifier sur [cette page](https://github.com/ACINQ/eclair/releases)) :  
`wget https://github.com/ACINQ/eclair/releases/download/v0.3.3/eclair-node-0.3.3-12ac145.jar`

* Les empreintes cryptographiques qui vont vous permettre de vérifier que le logiciel que vous avez téléchargé est bien identique à celui signé par les développeurs :  
`wget https://github.com/ACINQ/eclair/releases/download/v0.3.3/SHA256SUMS.asc`  

* La clé de Fabrice Drouin d'Acinq :  
```
wget https://acinq.co/pgp/drouinf.asc
gpg --import drouinf.asc
```

Nous pouvons alors vérifier l'origine et l'intégrité du fichier téléchargé :
* D'abord en vérifiant la signature de la release :  
```
$ gpg --verify SHA256SUMS.asc
gpg: Signature faite le ven. 31 janv. 2020 18:35:46 CET
gpg:                avec la clef RSA C25A288A842EAF7AA5B5303F7A73FE77DE2C4027
gpg: Bonne signature de « Fabrice Drouin <fabrice.drouin@acinq.fr> » [inconnu]
gpg: Attention : cette clef n'est pas certifiée avec une signature de confiance.
gpg:          Rien n'indique que la signature appartient à son propriétaire.
Empreinte de clef principale : C25A 288A 842E AF7A A5B5  303F 7A73 FE77 DE2C 4027
```

* Puis l'intégrité du fichier téléchargé avec cette commande :  
```
sha256sum --check SHA256SUMS.asc --ignore-missing
eclair-node-0.3.3-12ac145.jar: Réussi
sha256sum: Attention : 14 lignes ne sont pas correctement formatées
```

### Configuration de Bitcoin Core
Il va falloir modifier légèrement le fichier de configuration de Bitcoin Core.

Tout d'abord, arrêtez proprement Bitcoin Core :  
`bitcoin-cli stop`

Ouvrez le fichier `bitcoin.conf` (vous devriez commencer à avoir l'habitude maintenant). Nous allons modifier les deux lignes suivantes :  
```
zmqpubrawblock=tcp://127.0.0.1:28332
zmqpubrawtx=tcp://127.0.0.1:28332
```

Nous allons remplacer la valeur 28332 utilisée par lND avec 29000, qui correspond à Éclair. Concrètement :  
```
zmqpubrawblock=tcp://127.0.0.1:29000
zmqpubrawtx=tcp://127.0.0.1:29000
```

Enregistrez et fermez le fichier, puis relancez Bitcoin Core.

###Configuration d'Éclair

* Dans le dossier de l'utilisateur, créer le répertoire de configuration pour Éclair et le fichier de configuration :  
```
$ mkdir .eclair
$ nano .eclair/eclair.conf
```

Copier/coller le texte ci-dessous :
```
eclair.chain=mainnet
eclair.bitcoind.rpcport=8332
eclair.api.enabled=true
eclair.api.password=CHANGEMEORGETREKT //générez un password fort
eclair.bitcoind.rpcuser=foo //remplacez par la valeur correspondante dans votre fichier bitcoin.conf
eclair.bitcoind.rpcpassword=bar //remplacez par la valeur correspondante dans votre fichier bitcoin.conf
```

Enregistrez, puis quittez.

:point_right: toutes les options de configuration [ici](https://github.com/ACINQ/eclair/blob/master/eclair-core/src/main/resources/reference.conf).  

:point_right: Plus tard vous pourrez suivre les logs en tapant la commande suivante (`Ctrl-C` pour sortir) :  
`tail -f /home/bitcoin/.eclair/eclair.log`

### Lancez Éclair

Pour lancez Éclair, tapez simplement la commande suivante (si vous êtes dans le même dossier) :  
`java -jar eclair-node-0.3.3-12ac145.jar`

Éclair va alors se lancer, mais pas en arrière-plan. Afin d'éviter de bloquer votre terminal, je vous recommande de le lancer dans un terminal Screen.
```
screen -S eclair
java -jar eclair-node-0.3.3-12ac145.jar
```
Puis pressez `Ctrl + a`, puis `d` pour quitter Screen.

### Récupérer des bitcoins

Votre nœud Lightning est prêt. Contrairement à c-lightning et LND, Éclair a la particularité de ne pas avoir son propre wallet. Vous allez donc utiliser le wallet de Bitcoin Core pour ouvrir vos canaux.  
**ATTENTION: Éclair est une béta, Lightning une technologie expérimentale, ne mettez dessus que ce que vous pouvez vous permettre de perdre.**
* Générez une nouvelle adresse Bitcoin pour recevoir des fonds sur le portefeuille LND :  
`bitcoin-cli getnewaddress`  

* Envoyez des bitcoins sur cette adresse depuis n'importe quel wallet.  

* Vérifier le montant sur votre portefeuille.
`bitcoin-cli getbalance`  
  
### Installer eclair-cli
Il faudra attendre quelques confirmations pour que vos fonds soient accessibles sur le portefeuille. Vous pourrez alors ouvrir des canaux vers d'autres nœuds et effectuer des paiements à la vitesse de l'éclair !  
Profitez de ce répit pour installer eclair-cli, un petit exécutable qui permet de contrôler Éclair en ligne de commande.  

* Téléchargez `eclair-cli` :  
`wget https://raw.githubusercontent.com/ACINQ/eclair/master/eclair-core/eclair-cli`

* Copiez l'exécutable dans le répertoire ~/bin :  
`cp eclair-cli ~/bin`

* Ajoutez le mot de passe défini dans `eclair.conf` :  
```
nano ~/bin/eclair-cli
# décommentez cette ligne et collez le mot de passe dans eclair.api.password
api_password=<some-password>
```

Enregistrez et quittez. Testez la commande suivante :  
`eclair-cli getinfo`

:warning: Si cela ne marche, tapez `export PATH=$PATH:~/bin`, cela devrait résoudre le problème.

### Ouvrir des canaux de paiement

Il y a deux temps, d'abord il faut se connecter à un nœud, puis ensuite ouvrir un canal (vous pouvez regarder dans l'[explorer d'Acinq](https://explorer.acinq.co/) pour trouver quelques gros nœuds avec lesquels il serait intéressant de se connecter).  

Pour se connecter à un nœud, nous avons besoin d'un id (en fait, une clé publique) et d'une adresse IP (et parfois d'un numéro de port quand le standard 9735 n'est pas utilisé). Ces informations sont généralement présentées sous la forme <node-id>@<ip-adress>:<port>

* Se connecter :  
`eclair-cli --nodeId=<node-id> --host=<ip-adress> (--port=<port>)`

* Ouvrir un canal :  
`eclair-cli --nodeId=<node-id> --fundingSatoshis=<amount>`

Le deuxième argument est le montant avec lequel vous souhaitez initialiser le canal avec ce pair. Il sera déduit du montant disponible dans votre portefeuille. **Ce montant n'est pas exprimé en bitcoins, mais en satoshis**. Pour rappel, 1 btc == 100,000,000 sats.

:pointright: Toutes les commandes détaillées [ici](https://acinq.github.io/eclair/#introduction)

---

Voilà, il ne vous reste plus qu'à configurer Éclair avec [Tor](thunderbadger_69_tor.md) pour plus de confidentialité !

--- 

[ [Retour au Bonus](thunderbadger_60_bonus.md) ] ]
