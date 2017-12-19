Title: Diagnostiquer un problème de connexion Internet à La Réunion
Date: 2016-11-04T09:30:00+04:00
Category: misc

Je travaille à distance depuis La Réunion. La première condition pour
du télétravail réussi, c'est d'avoir <strong>une bonne connexion
Internet</strong>. C'est parfaitement possible à La Réunion, mais
comment faire quand ça ne va pas ? Il faut savoir où est un problème
pour pouvoir le corriger. En arrivant ici, j'aurais aimé lire un tel
article, alors je l'écris en espérant qu'il sera utile à d'autres.

En effet, les discussions sur la connexion Internet se concentrent
souvent sur les câbles sous-marins qui nous relient au réseau mondial.
C'est le cas par exemple d'un [rapport rendu à la préfecture en
décembre 2012][Rapport prix], certes plus focalisé sur la vie chère
que sur la qualité de la connexion.  Mais c'est aussi ce qui ressort
des nombreuses discussions que j'ai eues à ce sujet.  Pourtant, ces
câbles ne sont pas le gros du problème. Un cadre d'Orange avec qui
j'ai discuté dans un avion Paris - Réunion a d'ailleurs reconnu qu'ils
servaient souvent d'excuses. C'est dommage, parce qu'ils empêchent les
discussions constructives sur le sujet.

En résumé, voilà quoi faire quand vous avez un problème de connexion
ou une lenteur avec un des fournisseurs de l'île : SFR, Orange, Zeop,
Mediaserv, Canal Box ou Parabole Réunion.

## Débit, latence et ping

Il faut d'abord comprendre quelques caractéristiques simples mais
importantes d'une connexion Internet.

 1. La mesure la plus connue est le débit, annoncé en Mbit/s par les
    opérateurs.  Il mesure la vitesse à laquelle on peut récupérer un
    contenu, par exemple une vidéo : il doit être le plus grand
    possible.
 2. La latence (*ping* en anglais), bien connue des adeptes des jeux
    en ligne, mesure le temps qu'il faut pour obtenir un début de
    réponse. On la mesure en millisecondes : elle doit être la plus
    basse possible.
 3. Enfin, la gigue (*jitter* en anglais) mesure la variation de la
    latence, qui doit être la plus faible possible.

Pour mieux comprendre, on peut prendre l'image d'un robinet : le temps
que l'eau arrive après ouverture mesure la latence, la vitesse à
laquelle l'eau coule après la première goutte mesure le débit, et la
consistance avec laquelle l'eau coule mesure la gigue. Quand on ouvre
un robinet après avoir évacué l'eau des tuyaux, l'eau met du temps à
arriver (latence élevée) et arrive souvent par à-coups (gigue élevée)
jusqu'à que la situation se rétablisse.

En quoi ces mesures nous impactent ? Quand on visite un site web sur
Internet, notre navigateur fait des centaines de requêtes pour
récupérer les divers contenus de la page : texte mais surtout
publicités et images. Je viens d'essayer sur
[clicanoo.re](http://www.clicanoo.re/). La première fois, 279 requêtes
ont récupéré 2,9 Mo en 12 secondes. Au chargement suivant, 276
requêtes ont récupéré 0,4 Mo en 5 secondes (mon navigateur avait déjà
la plupart des informations).

Pourtant, le débit maximal mesuré par ma box est de 16 Mbit/s, soit
2 Mo/s, donc la première requête aurait pu terminer en 3 secondes, et
la deuxième en moins d'une seconde ! Ce qui prend du temps, c'est que
chaque requête à un nouveau serveur rajoute au moins 200 millisecondes
(0,2 secondes) au temps de chargement, et comme on ne peut pas toutes
les faire en même temps, ces petits délais s'ajoutent, et on finit par
avoir un site qui prend 10 secondes à charger entièrement.

Résumons. À La Réunion, les débits sont relativement élevés (bien !),
mais la latence l'est aussi du fait de l'éloignement géographique de
l'île (pas bien !). C'est acceptable tant que la gigue reste faible.
Mais dès que le réseau est un peu saturé, la latence et la gigue
s'envolent : il devient difficile de surfer sur Internet, et
impossible de discuter en vidéo (Skype par exemple).

## Identifier un problème

Mais comment savoir où est le problème ? Il peut être à différents
endroits :

  1. au niveau de l'ordinateur qui est trop lent, qu'il soit bourré de
     malware ou que son disque n'arrive pas à suivre
  2. au sein de la maison ou du bureau parce que les différents
     utilisateurs d'une même box saturent la Wi-Fi ou la connexion
     Internet
  3. à La Réunion parce que la capacité offerte à un quartier est
     insuffisante par rapport au nombre de lignes connectées
  4. entre La Réunion et la métropole parce qu'un câble sous-marin est
     endommagé (rare, mais [ça arrive][Coupure câble])
  5. en métropole ou ailleurs parce que le site distant est saturé

Le dernier point est facile à diagnostiquer : si un site est lent mais
pas les autres, alors c'est probablement le site qui a un problème,
pas vous. Dans les autres cas, que faire ?

Il y a une dernière notion à aborder avant de répondre à cette
question. La communication entre ordinateurs, par exemple entre vous
et un des serveurs de wikipedia.org, se fait par paquets : des bouts
de données suffisamment petits (de l'ordre du kilo-octet) pour
transiter rapidement. Un paquet va transiter par bonds, souvent une
bonne dizaine, avant d'arriver à sa destination. Le premier bond se
fait entre l'ordinateur et la box. (J'utilise "ordinateur" mais tout
ça est bien sûr applicable aux smartphones et tablettes.) Dans le cas
d'une connexion de Réunion à Réunion, il y a quelques bonds chez le
fournisseur d'accès à Internet (ou FAI, Orange dans mon cas), puis
peut-être quelques bonds chez un autre FAI suivant l'endroit où le
site est hébergé. Les outils classiques de tests d'une connexion
([Speedtest](https://en.wikipedia.org/wiki/Speedtest.net), ping) ne
descendent pas au niveau de ces bonds. Pour les voir, il faut des
outils plus fins, de type traceroute. Le plus pratique que j'ai trouvé
dans cette famille est [PingPlotter](https://www.pingplotter.com/),
une interface graphique bien faite (dommage, ce n'est pas un logiciel
libre, mais la version gratuite reste utile). Regardons le cas d'une
connexion à zeop.re qui est hébergé à La Réunion&nbsp;:

<img alt="Connexion à zeop.re" src="{filename}/images/pingplotter_zeop_small.png" width="100%" />

Bon, c'est quoi tout ça ? C'est un résumé visuel de ma connexion à
zeop.re toutes les 2,5 secondes sur 10 minutes. 

Commençons par le graphique du bas. Il montre la latence jusqu'à
destination dans le temps : on voit qu'on met entre 10 et 20
millisecondes, mais qu'il y a parfois des pics, le plus gros à la fin
d'environ 70 ms, et même une perte de connexion (la barre rouge).

La partie du haut contient d'abord des informations textuelles, à
gauche. Pour chaque étape (de 1 à 7), le nom ou l'adresse de la
machine qui s'est occupée de cette étape, et le temps moyen et minimum
pour un aller-retour jusqu'à là. Pour aller jusqu'à la dernière étape,
on met en moyenne 12,3 millisecondes, ce qui est plus précis que ce
qu'on avait lu graphiquement.

Enfin, il y a un autre graphique en haut à droite. En rouge, la
latence moyenne pour arriver à une étape donnée, en bleu les temps de
la dernière mesure, et en noir la latence minimum et maximum pour
atteindre une étape donnée. (Une [boîte à
moustache](https://fr.wikipedia.org/wiki/Bo%C3%AEte_%C3%A0_moustaches)
aurait été plus informative, tant pis.) Dans ce graphique, plus on va
à gauche, plus la latence est élevée. Normalement, on ne peut pas
"revenir en arrière" : comme chaque étape va plus loin, elle devrait
prendre plus de temps que l'étape précédente. Mais ce n'est pas ce qui
se passe entre les étapes 4 et 5. En fait, les [paquets envoyés pour
faire ces mesures](https://fr.wikipedia.org/wiki/Traceroute) sont
souvent traités avec une priorité plus faible, ce qui explique ces
anomalies.

En conclusion, ici tout va bien, la latence bouge peu au cours du
temps, et elle est d'ailleurs très faible, parce qu'on ne va pas
loin&nbsp;: on reste à La Réunion. Regardons maintenant un accès à un site
hébergé en métropole&nbsp;:

<img alt="Connexion à orange.re" src="{filename}/images/pingplotter_orange_normal_small.png" width="100%" />

Et oui, orange.re est hébergé en métropole ! Tout comme reunion.fr et
tant d'autres. (C'est parce que c'est moins cher.) Bon, qu'est-ce que
ça donne ? La latence est plus élevée (supérieure à 200
millisecondes), et il y a plus de bonds. Mais tout a l'air de bien se
passer : la latence reste stable.

Rentrons dans le détail. Entre les étapes 3 et 4, on voit très
clairement le paquet qui "saute la mer", comme on dit en créole
réunionnais. C'est ce qui prend le plus gros du temps (194
millisecondes) : on comprend que ce sera difficile d'avoir une latence
sous les 200 millisecondes pour accéder à un site en Europe.
Maintenant qu'on comprend le cas normal, essayons de voir ce qui se
passe quand ça ne va pas. Il y a un an, tous les soirs le réseau était
inutilisable chez moi, mais depuis des travaux réalisés par Orange
tout se passe à merveille.  Je ne peux donc que simuler un problème au
niveau de mon propre réseau, en saturant ma connexion Internet. Voilà
ce que ça donne&nbsp;:

<img alt="Connexion à orange.re avec ADSL saturé" src="{filename}/images/pingplotter_orange_adsl_sature_small.png" width="100%" />

Ici, les perturbations commencent dès le deuxième bond, ce qui permet
de comprendre l'origine du problème : la connexion de ma box au réseau
Orange. Sur une interface qui n'affiche que des latences, on peut
diagnostiquer des problème de débit : ici j'ai atteint le débit
maximal offert par ma connexion. Le câble sous-marin lui n'a rien à
voir, les paquets prennent toujours le même temps pour traverser.
Voyons un autre problème&nbsp;:

<img alt="Connexion à orange.re avec WiFi saturé" src="{filename}/images/pingplotter_orange_box_saturee_small.png" width="100%" />

Ici, il y a des problème dès le premier bond : c'est la Wi-Fi qui est
saturée ! Et en effet, je transférais un gros fichier entre deux
ordinateurs via la box, qui était le goulot d'étranglement,
c'est-à-dire le composant à améliorer pour que le transfert soit plus
rapide. Enfin, un dernier exemple&nbsp;:

<img alt="Connexion à wikipedia.org" src="{filename}/images/pingplotter_wikipedia_normal_small.png" width="100%" />

Ici, il faut 19 bonds, dont un pour aller d'abord en métropole, et un
autre pour traverser l'Atlantique jusqu'à wikipedia.org. Ce sont ces
deux gros bonds qui expliquent la latence moyenne proche de 400ms.

## À quoi ça sert ?

Depuis un an et demi, j'ai assisté à toute une série de problèmes.
Dans 95% des cas, les câbles n'étaient pas en cause. Mais ce qui est
le plus agréable, c'est que j'étais toujours capable de dire à quel
niveau était le problème. Sans cette information, on ne peut pas
envisager de le résoudre. Si c'est un problème local, tant mieux, mais
sinon on est bien armé pour contacter le deuxième niveau de support de
son fournisseur d'accès.

Aujourd'hui, je ne peux plus dire que ma connexion Internet me pose un
problème. J'ai encore quelques coupures lors des fortes pluies : les
câbles et équipements du réseau ne résistent pas toujours longtemps à
l'humidité et prennent parfois l'eau. Là, il n'y a pas grand chose à
faire : je perds complètement la synchronisation ADSL. Ceci dit, le
fait de pouvoir documenter les coupures nous arme au moment de
contacter notre FAI, qui finit par agir. C'est comme ça qu'un
technicien est venu remplacer un câble oxydé chez moi, ce qui a
sensiblement augmenté mon débit et a réduit les coupures causées par
la pluie.

La suite ? J'espère être éligible à la fibre d'ici quelques mois. Je
pourrai alors faire une comparaison détaillée et appuyer mon ressenti
de vitesse avec des chiffres précis.

Merci à ma femme Élodie ainsi qu'à Florent Bouckenooghe pour leurs
relectures et conseils avisés. Les erreurs sont miennes.

[Jeff Atwood]: http://firstround.com/review/Heres-Why-Youre-Not-Hiring-the-Best-and-the-Brightest/
[Rapport prix]: http://administration.reunion.gouv.fr/spip.php?article1669
[Coupure câble]: http://www.ipreunion.com/actualites-reunion/reportage/2016/09/03/internet-si-la-connexion-est-lente-c-est-peut-etre-a-cause-du-safe,49318.html

<!-- vim: spelllang=fr
-->
