Title: Connexion Internet à la Réunion : les vrais défis
Date: 2016-11-04T09:30:00+04:00
Category: misc

J'ai emménagé l'an dernier à l'île de la Réunion, où j'ai la chance de
travailler à distance pour [Clustree](https://www.clustree.com/) en
tant que "développeur back-end". Comme le [dit bien Jeff Atwood][Jeff
Atwood], la seule condition à respecter pour le télétravail, c'est
d'avoir <strong>une bonne connexion Internet</strong>. Mais est-ce
possible à la Réunion ? Et que faire quand ça ne va pas ? Et comment
savoir ce qui ne va pas ? En arrivant à la Réunion, j'aurais aimé lire
un tel article, alors je l'écris en espérant qu'il sera utile à
d'autres.

En effet, les discussions sur la connexion Internet à La Réunion se
concentrent souvent sur les câbles sous-marins qui nous relient au
réseau mondial. C'est le cas par exemple d'un [rapport rendu à la
préfecture en décembre 2012][Rapport prix], certes plus focalisé sur
la vie chère que sur la qualité de la connexion. Mais c'est aussi ce
qui ressort des nombreuses discussions que j'ai eues à ce sujet.
Pourtant, ces câbles ne sont pas le gros du problème. Un cadre
d'Orange avec qui j'ai discuté dans un avion Paris-Réunion a
d'ailleurs reconnu qu'ils servaient souvent d'excuses.

## Débit, latence et ping

Il faut d'abord mieux comprendre quelques caractéristiques simples
mais importantes d'une connexion Internet :

 1. La mesure la plus connue est le débit, annoncé en Mbit/s par les
    opérateurs.  Il mesure la vitesse à laquelle on peut récupérer un
    contenu, par exemple une vidéo : il doit être le plus grand
    possible.
 2. La latence (*ping* en anglais), bien connue des adeptes du jeu en
    ligne, mesure le temps qu'il faut pour obtenir un début de
    réponse, on le mesure en millisecondes : il doit être le plus bas
    possible.
 3. Enfin, la gigue (*jitter* en anglais) mesure la variation de la
    latence, qui doit être la plus faible possible. Pour les
    discussions vidéo (Skype par exemple), une latence même faible
    mais qui varie beaucoup rendra la discussion impossible.

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

À la Réunion, les débits sont relativement élevés (j'attends quand
même que la fibre arrive jusqu'à chez moi !), mais la latence l'est
aussi. C'est acceptable tant que la gigue reste faible. Mais dès que
le réseau est un peu saturé, la gigue s'envole, il devient difficile
de surfer sur Internet, et impossible de discuter en vidéo par Skype,
Facetime ou autres Hangouts.

## Identifier un problème

Mais comment savoir où est le problème ? Il peut être à différents
endroits :

  1. au sein de la maison ou du bureau parce que les différents
     utilisateurs d'une même box saturent la connexion
  2. à la Réunion parce que la capacité offerte à un quartier est
     insuffisante par rapport au nombre de lignes connectées
  3. entre la Réunion et la métropole parce qu'un câble sous-marin est
     endommagé (rare, mais [ça arrive][Coupure câble])
  4. en métropole ou ailleurs parce que le site distant est saturé

Le dernier point est facile à diagnostiquer : si un site est lent mais
pas les autres, alors c'est probablement le site qui a un problème,
pas vous. Dans les autres cas, que faire ?

Il y a une dernière notion à aborder avant de répondre à cette
question. La communication entre ordinateurs (par exemple entre vous
et un des serveurs de wikipedia.org) se fait par paquets : des petits
bouts de données suffisamment petit pour transiter rapidement. Un
paquet va transiter par bonds, souvent une bonne dizaine, avant
d'arriver à sa destination. Le premier bond se fait entre l'ordinateur
et la box. Dans le cas d'une connexion de Réunion à Réunion, il y a
quelques bonds chez le FAI (Orange dans mon cas), puis peut-être
quelques bonds chez un autre FAI suivant l'endroit où le site est
hébergé. Les outils classiques de tests d'une connexion (speedtest,
ping) ne descendent pas à ce niveau. Pour voir les bonds, il faut des
outils plus fins, de type traceroute. Le plus pratique que j'ai trouvé
dans cette famille est [PingPlotter](https://www.pingplotter.com/),
une interface graphique bien faite (dommage, ce n'est pas un logiciel
libre, mais la version gratuite reste utile). Regardons le cas d'une
connexion à zeop.re qui est hébergé à la Réunion :

<img alt="Connexion à zeop.re" src="{filename}/images/pingplotter_zeop_small.png" width="100%" />

Ici, il y a 7 bonds, qui prennent en moyenne 12 millisecondes avant
d'arriver à destination. Il y a un peu de bruit, qui donne
l'impression qu'on arrive à la cinquième étape 5 millisecondes avant
d'arriver à la quatrième étape, mais de toute façon ici c'est
tellement rapide que l'analyse détaillée n'est pas utile. Regardons
maintenant un accès à un site hébergé en métropole&nbsp;:

<img alt="Connexion à orange.re" src="{filename}/images/pingplotter_orange_normal_small.png" width="100%" />

Entre les étapes 3 et 4, on voit très clairement le paquet qui "saute
la mer", comme on dit en créole réunionnais. C'est ce qui prend le
plus gros du temps (194 millisecondes) : ce sera difficile d'avoir une
latence sous les 200 millisecondes pour accéder à un site en Europe.
Maintenant qu'on comprend le cas normal, essayons de voir ce qui se
passe quand ça ne va pas. Il y a un an, tous les soirs le réseau était
inutilisable, mais depuis des travaux réalisés par Orange tout se
passe à merveille.  Je ne peux donc que simuler un problème au niveau
de mon propre réseau, en saturant ma connexion Internet. Voilà ce que
ça donne&nbsp;:

<img alt="Connexion à orange.re avec ADSL saturé" src="{filename}/images/pingplotter_orange_adsl_sature_small.png" width="100%" />

Ici, les perturbations commencent dès le deuxième bond, ce qui permet
de comprendre l'origine du problème : la connexion de ma box au réseau
Orange. Sur une interface qui n'affiche que des latences, on peut
diagnostiquer des problème de débit : ici j'ai atteint le débit
maximal offert par ma connexion. Le câble lui n'a rien à voir, les
paquets prennent toujours le même temps pour traverser. Voyons un
autre problème&nbsp;:

<img alt="Connexion à orange.re avec WiFi saturé" src="{filename}/images/pingplotter_orange_box_saturee_small.png" width="100%" />

Ici, il y a des problème dès le premier bond : c'est la Wi-Fi qui est
saturée ! Et en effet, je transférais un gros fichier entre deux
ordinateurs via la box, qui était le goulot d'étranglement,
c'est-à-dire le composant à améliorer pour que le transfert soit plus
rapide. Enfin, un dernier exemple&nbsp;:

<img alt="Connexion à wikipedia.org" src="{filename}/images/pingplotter_wikipedia_normal_small.png" width="100%" />

Ici, il faut 19 bonds pour aller d'abord en métropole, puis pour
traverser l'Atlantique. Ces deux gros bonds sont très marqués, et
expliquent la latence moyenne proche de 400ms.

## À quoi ça sert ?

Depuis un an et demi, j'ai assisté à toute une série de problèmes.
Dans 95% des cas, les câbles n'étaient pas en cause. Mais ce qui est
le plus agréable, c'est que j'étais toujours capable de dire à quel
niveau était le problème, ce qui est très utile pour qu'un contact
avec le support niveau 2 de mon FAI soit fructueux.

Aujourd'hui, je ne peux plus dire que ma connexion Internet me pose un
problème. J'ai encore quelques coupures lors des fortes pluies : les
câbles et équipements ne résistent pas toujours longtemps à l'humidité
et prennent parfois l'eau. Là, il n'y a pas grand chose à faire : je
perds complètement la synchronisation ADSL. Ceci dit, le fait de
pouvoir documenter les coupures permet d'être armé au moment de
contacter son FAI, qui finit par agir si les coupures sont trop
nombreuses.

La suite ? J'espère être éligible à la fibre d'ici quelques mois. Je
pourrai faire une comparaison détaillé et appuyer mon ressenti de
vitesse avec des chiffres précis.

[Jeff Atwood]: http://firstround.com/review/Heres-Why-Youre-Not-Hiring-the-Best-and-the-Brightest/
[Rapport prix]: http://administration.reunion.gouv.fr/spip.php?article1669
[Coupure câble]: http://www.ipreunion.com/actualites-reunion/reportage/2016/09/03/internet-si-la-connexion-est-lente-c-est-peut-etre-a-cause-du-safe,49318.html
