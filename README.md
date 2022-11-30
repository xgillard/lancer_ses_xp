# Best Practices for Running Experiences in INGI

Le but de cette page est de regrouper et expliquer les bonnes pratiques pour
faire tourner des expériences sur l'infrastructure INGI.

## Réserver une machine et s'y connecter

Quand on lance des XP, c'est souvent pour faire tourner plein de benchmarks 
avec plein de configurations différentes... et ca prend enormément de temps 
de calcul. Il n'est donc pas question de lancer tout ca sur son laptop et 
attendre 2 mois que les XP aient fini de tourner.  Il faut donc lancer ca
sur une des machines de l'insfrastructure INGI.

### Prerequis

>> Avant meme de lancer des expériences sur une machine de l'infrastructure, il
>> est utile d'avoir une clef SSH et s'assurer qu'elle soit bien installée sur
>> les passerelles (Si ce que je raconte n'est pas encore clair... ca va
>> venir). C'est une operation qu'il ne faut faire qu'une et une seule fois. 

#### Creation d'une clef ssh

Dans un terminal il faut lancer la commande suivante.
Il va vous poser quelques questions, auxquelles il suffit de répondre **MAIS**
au moment ou il va vous demander un _passphrase_ le mieux est de ne pas mettre
de pass phrase du tout. L'intéret de ne pas en mettre, c'est que vous pourrez
vous connecter directement sur les machines de calcul sans avoir besoin de 
taper 15 fois votre mot de passe.

```
ssh-keygen -t rsa 
```

Le résultat de cette commande, c'est qu'un dossier `.ssh` a été crée dans votre
home folder. Ce dossier contient deux fichiers: 
- `id_rsa` qui est votre clef privée (et que vous ne devez *jamais* partager)
- `id_rsa.pub` qui est votre clef publique et que vous pouvez partager sans
  probleme.


#### Installation de la clef ssh sur les machines distantes

Une fois que vous avez cree votre clef ssh (privée et publique), vous devez 
vous assurer qu'elles soient bien installés sur les machines de passerelles
et sur les machines de calcul. Concretement, ca veut dire que vous devez 
envoyer un mail aux sysadmins 'request-ingi' at 'uclouvain' point 'be' en leur
expliquant ce que vous voulez faire, et en leur joignant votre fichier
`id_rsa.pub` en piece jointe. 

Pour que la configuration soit complete, vous allez devoir aussi copier vos
clef privee (`id_rsa`) et publique (`id_rsa.pub`) dans votre home folder (vous
allez devoir creer les fichiers `~/.ssh/id_rsa` et `~/.ssh/id_rsa.pub` sur
`ssh1.info.ucl.ac.be` et ajouter le contenu de votre clef publique dans le
fichier `~/.ssh/authorized_keys`) sur la machine de passerelle
(`ssh1.info.ucl.ac.be`). 

Si vous rencontrez des problemes, contactez les admin. Ils vous aideront et 
vous guideront.

##### Validation

Pour valider que votre installation est ok, tapez la commande suivante dans un
terminal sur votre machine. Ca devrait vous permettre de vous connecter a la
passerelle `ssh.info.ucl.ac.be` sans avoir a taper votre mot de passe.

```
LOCAL  !!> hostname
LOCAL  !!> <ca repond le nom de votre laptop>
LOCAL  !!> ssh ssh.info.ucl.ac.be
REMOTE $$> hostname
REMOTE $$> <ca repond ssh.info.ucl.ac.be>
```

### Réservation d'une machine

Afin de pouvoir lancer des expériences sur une machine de l'infrastructure, il
faut d'abord que vous réserviez une machine sur laquelle vous allez pouvoir
lancer les XP. Pour cela, vous devez vous rendre sur
`https://reservation.info.ucl.ac.be`. 

Sur cette page, il suffit de vous connecter avec votre login INGI (si vous n'en
avez pas il va de nouveau faloir embeter les sysadmins). Puis vous allez pouvoir
sélectionner une machine (menu a gauche) et un moment a partir duquel commencer
la réservation (calendrier du milieu qui prend toute la place).

Pour creer une reservation, il suffit de cliquer sur le signe "+" dans la time
line d'une machine puis remplir le petit formulaire.

**VOILA, VOUS AVEZ RESERVE UNE MACHINE POUR FAIRE TOURNER VOS TRUCS**. Vous
allez bientot pouvoir vous y connecter en ssh mais il faut généralement attendre
un petit peu avant d'y avoir effectivement acces. (Donc si vous essayez de
vous connecter en SSH et que ca ne marche pas... patience: ca peut prendre
jusqu'a 30 minutes pour qu'une réservation soit effective).


### Se connecter 

Pour se connecter a la machine, le plus simple est d'utiliser la commande 
suivante. Celle-ci va en fait faire deux choses:
1. elle va établir une connection ssh depuis votre laptop vers la passerelle
2. depuis la passerelle, etablir une connection ssh vers la machine réservée.

C'est pratique car cela vous permet d'accéder aux machines de calcul depuis
partout dans le monde comme si vous étiez sur le LAN en INGI.

```
ssh -J ssh.info.ucl.ac.be <lamachine_reservee>
```

Par exemple si vous avez réservé le serveur "wauters", vous allez taper la
commande
```
ssh -J ssh.info.ucl.ac.be wauters.info.ucl.ac.be
```


## Charger son code sur la machine réservée

Le plus clean que vous puissiez faire pour lancer vos XP est simplement de
mettre tout votre code sur un repository git (pour moi c'est sur github) puis
de cloner ce repo sur la machine que vous avez reserve.

**IMPORTANT**
>> Les machines que vous réservez ont en fait 2 file systems qui sont montés.
>> Le premier est un file system réseau qui est partagé entre la plupart des
>> serveurs linux du réseau INGI. C'est tres pratique mais ca veut aussi dire
>> que ce filesystem est tres **LENT** !
>>
>> Quand vous clonez ou compilez du code, assurez vous donc toujours que toutes
>> les opérations se passent sur le 2e filesystem. En effet, le 2e filesystem
>> est un file system local et est -- lui -- tres rapide. Pour accéder a ce
>> second filesystem, vous allez devoir vous rendre dans le dossier 
>> `/home/<votre_username>`. Si vous n'avez pas acces a ce dossier, il suffit
>> de le creer.


## Compiler son code

Comme il n'y a pas d'interface graphique sur le server a distance, il n'est
pas possible de compiler votre code pour en faire un executable en cliquant 
un peu partout dans votre IDE. Il faut des lors utiliser un buildsystem pour
creer les fichiers binaires.


### Si votre code est en Python

Il n'y a rien a compiler donc rien a faire. Il faudra peut etre quand meme 
installer les dépendances manquantes dans un virtualenv.

### Si votre code est en Rust

Tout va se passer exactement comme sur votre laptop, et vous allez creer le 
binaire en utilisant cargo. La toute premiere fois, il y a fort a parier que
cargo ne soit pas installé sur la machine ou vous vous trouvez. Il faudra donc
l'installer et ajouter la toolchain rust a votre path.

Pour installer rustc et cargo sur le server, il suffit de lancer la commande
suivante et puis configurer votre PATH.
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Si la version qui est installée n'est pas suffisamment a jour a votre gout, 
vous pourrez la mettre a jour via 
```
rustup update
```

### Si votre code est en Java

Si vous avez décidé d'écrire votre code en Java, le mieux est d'utiliser
**Maven** pour compiler votre code. Comme java et maven ne sont pas installés
globalement pour tout le monde sur les machines, vous allez devoir les installer
vous meme en suivant les instructions qui sont données ici:

- https://docs.oracle.com/javase/8/docs/technotes/guides/install/linux_jdk.html
- https://maven.apache.org/install.html

**NOTE**
>> Comme vous n'avez pas d'interface graphique sur le server, vous n'avez pas
>> non plus de navigateur web pour aller telecharger le dernier JDK et Maven.
>> Alors comment les télécharger sur le server ? 
>>
>> C'est tres simple: il suffit d'utiliser la commande `wget` ou `curl` 
>> de telecharger un fichier depuis la ligne de commande.
>>
>> Par exemple, pour telecharger la derniere version du JDK et la derniere 
>> version de maven, j'aurais tapé les commandes suivantes:
>>
>> ```
>> wget https://download.oracle.com/java/19/latest/jdk-19_linux-aarch64_bin.tar.gz 
>> wget https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz
>> ```


## Executer les XP en parallele

TODO (gnu parallel)

## Eviter que ca plante lorsqu'on se deconnecte

TODO (screen ou nohup)

## Recuperer les résultats

TODO

## Analyser les résultats

TODO (nushell ou pandas)

## Visualiser les résultats

TODO (gnuplot ou matplotlib)


