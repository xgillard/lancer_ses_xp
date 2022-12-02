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
```
wget https://download.oracle.com/java/19/latest/jdk-19_linux-aarch64_bin.tar.gz 
wget https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz
```


## Executer les XP en parallele

Une grosse part de l'interet d'utiliser les machines de calcul de 
l'infrastructure INGI vient du fait que ce sont de grosses machines avec pas
mal de ressources disponibles. Ce qui veut dire que si vous pouvez en profiter
pour faire tourner plein d'xp a la fois plutot que de les faire tourner une
apres l'autre. Pour ca, vous allez utiliser la commande gnu parallel. Parallel
n'est pas installée par défaut sur les machines, donc vous allez devoir faire
l'installation vous meme. 

```
wget http://ftp.gnu.org/gnu/parallel/parallel-20120122.tar.bz2
tar -xpf parallel-20120122.tar.bz2
cd parallel-20120122
./configure && make && make install
```

### NOTE
Quand on lance des sets d'xp, le mieux est toujours d'avoir un programme qui 
accepte un fichier d'instance en entrée pour effectuer l'expérience sur cette
seule instance. Idealement ce programme ecrit ses resultats sur la sortie 
standard dans un format qui utilise un séparateur. Typiquement, chaque xp va
ecrire une ligne au format CSV (personnellement j'ai tendance a utiliser le
séparateur '|' dans mes outputs et a faire en sorte que tout soit toujours bien
aligné. C'est sans doute pcq je suis un peu maniaque a ce niveau la, mais aussi
pcq j'aime bien de pouvoir facilement lire les resultats des xp en cours pour
pouvoir tout arreter si je vois que qqch se passe mal).

Pour donner un example, en general ca ressemble a qqch comme ca chez moi:
```
./programme_xp instance_001.txt
<nom de l'instance> | <nom de la config utilisee pour l xp> | <status: proved ou timeout> | <best objective> | <lower bound> | <upper bound> | <temps d'exec en secondes> | <solution>
```


### Utiliser parallel pour paralleliser des taches

Une fois que `parallel` est installé, c'est tres facile a utiliser. Si on 
suppose que lancer le programme `truc <fichier_instance>` permet de lancer une
expérience pour une instance de benchmark comme expliqué au dessus, et que le
dossier `benchmarks` contient toutes les instances qu'on veut tester; alors
on peut lancer toutes les instances en parallele avec la commande suivante:

```
find ./benchmarks -type f -name \*.instance | parallel -I% ./truc % >> resultat.txt 
```

La commande ci-dessus fait pas mal de trucs, et elle devrait grosso modo etre
au coeur de vos scripts qui font tourner vos xp. On va donc detailler ce qu'elle
fait:

* `find ./benchmarks -type f -name \*.instance`  liste tous les fichiers qui 
   se trouvent dans le répertoire "benchmarks" (et sous repertoires) et dont
   l'extension de fichier est ".instance". En fonction des benches avec lesquels
   vous travaillez, vous allez vouloir inclure ou non le flag `-name ...` pour
   limiter les fichiers qui sont listés (la commande marche aussi sans ca).
   Si vous voulez en savoir plus sur la commande `find`, le mieux est d'aller
   lire la page de manuel. C'est un peu long, mais ca contient enormement d'info
   utiles. (Pour lire la page de manuel, il faut taper la commande `man find`. 
   Si vous etes sur un systeme windows, on trouve en general la meme page
   si on tape "man find" dans google... il y en a qui n'aiment pas lire dans le
   terminal).

* `parallel -I% ./truc %` dit que pour chacun des fichiers qui sont recus sur
   l'entrée standard (a travers le pipe qu'on a mis entre les commandes `find`
   et `parallel`), on va lancer une instance du programme `truc` en lui passant
   chaque fois un fichier en entree. Le flag `-I%` que j'ai utilisé sert a 
   indiquer que dans la commande "generique" qu'on est en train d'ecrire, le
   placeholder "%" sert a denoter le fichier d'input qui nous est transmis par
   find.

* `>> resultat.txt` indique qu'on veut rediriger les resultats de chacun des
   programmes qui ont etes executés et concatener les lignes qu'ils ont produit
   dans un fichier qui s'appelle 'resultats.txt'

#### ATTENTION

Si vous lancez vos xp exactement comme je viens de l'expliquer, vous allez avoir
un petit probleme: lorque vous allez fermer la connexion ssh utilisée pour
lancer vos xp, le systeme d'exploitation va penser que le processus peut etre
tué vu que de toutes facons vous n'etes plus la pour le regarder tourner.
(Techniquement le systeme va envoyer le signal SIGHUP a votre programme... ce
qui aura comme effet de le faire planter)

La solution consiste donc a empecher le signal SIGHUP de venir terminer votre 
code et il y a deux solutions pour ca. La premiere est tres facile a mettre
en place mais est un petit peu moins pratique si vous vous rendez compte que
vos xp sont parties en vrille et qu'il faut les tuer. La deuxieme approche
ne pose pas ce probleme, mais je la connais moins bien et ne l'ai encore jamais
utilisée (ca serait cool si qqn pouvait completer la section).

##### 1e solution: nohup
La premiere solution consiste a utiliser la commande `nohup` qui est installee
sur le systeme puis a lancer le processus en arriere plan. Comme son nom 
l'indique, `nohup` sert a bloquer le signal SIGHUP et a l'empecher d'atteindre
votre programme. 

Concretement, si on repart de l'exemple d'au dessus, la commande deviendrait
```
nohup find ./benchmarks -type f -name \*.instance | parallel -I% ./truc % >> resultat.txt  &
```

Le `nohup` placé en tout debut de ligne permet de bloquer le signal, et le `&`
placé tout a la fin, sert a dire qu'il faut lancer le processus en arriere plan.
Comme ca, vous pourrez tranquillement quitter le shell sans que vos xp ne 
s'arretent ET vous allez pouvoir voir l'etat d'avancement du truc en regardant
le contenu du fichier de resultats.


##### 2e solution: screen
La 2e solution consiste a utiliser la commande `screen`. Celle-ci permet de 
faire croire a l'OS qu'on est toujours en train de visualiser le processus qui
tourne via un "ecran virtuel". L'avantage, c'est qu'on peut decider de se
decrocher de cet ecran virtuel sans que l'OS ne pense pour autant qu'on a arrete
de regarder le processus tourner. Par ailleurs, si on tue le process `screen`
ca va avoir pour effet que l'OS va envoyer le signal SIGHUP a toutes les xp qui
sont en train de tourner, et faire en sorte que tout s'arrete proprement.


## Recuperer les résultats

Pour recuperer vos resultats, vous pouvez utiliser la commande `scp` qui permet
de copier des fichiers a distance. C'est le plus facile, mais malheureusement
ce n'est pas toujours faisable (le port pourrait ne pas etre ouvert etc...).
Du coup, ma technique puor aller recuperer mes resultats facilement et qui marche
toujours est la suivante.

Si je sais que mon fichier de resultats se trouve en `/home/xgillard/xptruc/resultats.txt`
je vais pouvoir les recuperer directement sur mon pc avec la commande suivante:
```
ssh -J ssh1.info.ucl.ac.be <machine_reservee>.info.ucl.ac.be "cat /home/xgillard/xptruc/resultats.txt" > resultats.txt
```

Je comprend que la commande au dessus puisse avoir l'air un peu barbare, donc
on va la decouper en petits morceaux:

* `ssh -J ssh1.info.ucl.ac.be <machine_reservee>.info.ucl.ac.be` c'est exactement
  la meme commande qu'au tout debut de ce document: on se connecte a la machine
  reservee en faisant un saut de puce via le serveur `ssh1.info.ucl.ac.be`. 
  La difference c'est que cette fois, on a rajouté `"cat /home/xgillard/xptruc/resultats.txt"`
  apres la connexion ssh. De ce fait, ssh ne va pas nous lancer une session
  interactive comme on en a toujours eu jusqu'ici; mais il va plutot executer
  la commande `cat /home/xgillard/xptruc/resultats.txt` a distance. Cette
  commande sert tout simplement a imprimer le contenu du fichier sur la sortie
  standard (qui nous est transmise par ssh).

* Apres la commande ssh, on peut voir que le resultat de l'execution a distance
  est redirigée dans un fichier qui s'appelle "resultats.txt" comme sur 
  la machine distante grace a `> resultats.txt`. 


## Analyser les résultats

Concernant l'analyse des résultats, le plus simple est soit d'utiliser pandas
soit d'utiliser nushell. Si vous etes un amoureux du python, vous allez peut
etre vouloir vous diriger vers pandas. Et si vous ne le connaissez pas, vous
voudrez sans doute lire ce tutoriel:
https://www.w3schools.com/python/pandas/default.asp

Si par contre vous etes un peu plus aventureux, vous allez peut etre vouloir
installer et utiliser nushell (https://www.nushell.sh/). Leur tutoriel est 
assez court et super complet; ca se lit tres facilement:
https://www.nushell.sh/book/

Personellement, ma preference va a nushell que j'ai trouvé plus ergonomique
(car ca combine pas trop mal algebre relationelle et script shell). Par ailleurs, 
mais c'est un peu la cerise sur le gateau, nushell permet de produire les
resultats beaucoup plus rapidement que le couple python + pandas. Au final,
c'est surtout une histoire de gout.. donc testez peut etre les deux et voyez
ce qui marche le mieux pour vous.

## Visualiser les résultat

TODO (gnuplot, matplotlib, excel)


