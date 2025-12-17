# Rapport déploiement de réseaux
## Alexandre Vanicotte--Hochman, Arthur Bongiovanni

### 1.Introduction
AFDX est un super protocole que vous conaissez au moins aussi bien que nous. Dans ce projet, l'objectif est d'utiliser DPDK pour changer les règles de commutation d'un switch afdx simmulé et introduire donc du vl forwarding puis du vl multicasting afin de calculer des délais pires cas (le délai moyen ne nous interesse pas puisqu'on cherche a certifier avec le pire cas).
pour le protocole opératoire, nous branchons 2 pc de salle c308. 
le pc 1 servira de simulateur de traffic a l'aide d'un script python et d'un commande bash qui permet d'envoyer les trames dés que possible. Ce script enverra de façon altérnée des messages sur le vl 1 et 2.
```bash
for i in {1..2000};do
sudo python send.py;
done
```
Le pc 2 servira de switch et erst équipée de 2 ports parrallèles.
on relie les interfaces eth2 et eth3 de la machine 1 aux ports parrallèles de la machine 2  selon la logique suivante avec des cables rj45:
```
eth3<->port 0
eth2<->port 1 
```

### 2.VL forwarding

Dans ce premier cas, on active la macro
```c
//#define MCASTVLFORWARD
 #define VLFORWARD
// #define FORWARD
```
la table de comutation [](route_vl-fwd.txt) est un fichier texte dont les lignes sont de cette forme:
`vl port`
On obtient au bout de 2000 trames envoyés vers le port 0: 
D $_{commutation~max~port~0}$=351ns
D $_{commutation~max~port~1}$=705ns

on obtient donc des valeurs bien inférieures a 8µs (délai de certification).

### 3.VL MultiCast
Dans ce second cas, les 2000 trames sont envoyés vers le port 0, encore une fois alternativement sur vl 1 et 2.

la table de comutation [](route_vl-mcast.txt) est un fichier texte dont les lignes sont de cette forme:
`vl port1 port2 .... port n`

On obtient au bout de 2000 trames envoyés vers le port 0: 
D $_{commutation~max~port~0}$=1500ns
D $_{commutation~max~port~1}$=1047ns

on obtient donc aussi des valeurs bien inférieures a 8µs (délai de certification).

On observe une inversion des maxima dans le cas de l'inversion de l'ordre dans la table de commutation. En effet, la structure est un tableau de liste chainées, la tête est alors la fin de la ligne sur la table de commutation.
### 4.Critique de la méthode
Notre méthodologie met en évidence des délais largements satisfaisant. Le BAG permettrait d'espacer les envois de trames, et donc d'aviter des drop et des saturations coté switch. Notre démo montre toutefois que avec un débit continu, les paquets sont bien transmis au niveau du switch, et donc que le bag ne pourrait qu'ameliorer la situation.
Fun Fact: au départ, le port 1 n'étais pas physiquement connécté, le buffeur d'envoi se sature alors avec 542 paquets. 

### 5.Conclusion
Nous avons donc programmé un switch qui pourrait être certifié selon nos observations.