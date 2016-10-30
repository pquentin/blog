Title: This is the end?
Date: 2010-02-09T14:07:35+01:00
Category: Logiciel libre

**Note été 2014 :** Ce billet a été écrit en février 2010, suite à mon Google
Summer of Code de l'été 2009. Depuis, tout le fantastique travail sur BZFlag 3
(ainsi que l'argent qui a été investi) a été jeté et certains développeurs
travaillent calmement sur BZFlag 2.4 (la version actuelle) et [BZFlag
2.6](https://github.com/BZFlag-Dev/bzflag-import-3/tree/v2_6_x) (la prochaine
version). C'était la meilleure solution.

**Note :** Je regrette aussi le ton un peu abrasif.

BZFlag est un jeu assez connu sous GNU/Linux du fait de son ancienneté, sa
portabilité, sa stabilité, et son gameplay assez original. J'ai eu la chance de
participer à son développement l'été dernier, pendant le Google Summer of Code,
et je voulais en profiter pour parler de ce projet, de son fonctionnement
interne, et des erreurs qui ont été commises quand à sa gestion. Je pensais
avant d'écrire  que cette analyse pouvait être bénéfique aux contributeurs dans
le libre de manière générale qui peuvent avoir envie d'apprendre des erreurs
des autres. Finalement, pas tant que ça, les erreurs faites sont assez
évidentes. Je ne prétend pas toute fois donner des leçons à BZFlag. Ce billet
est au plus une occasion de m'expliquer pourquoi j'ai arrêté de contribuer à ce
projet qu'autre chose. Je ne suis pas là depuis si longtemps (environ un an),
et je m'inquiète peut-être pour rien.

## L'histoire

Je vais commencer par quelques stats et quelques faits que je connaissais pas
avant de rentrer dans l'équipe. Rien de précis, c'est pour donner une idée.
BZFlag a environ 80 développeurs qui ont encore le droit de commit sur
sourceforge, a commencé son développement il a environ 15 ans, a été libéré en
97. Ça a attiré beaucoup de gens, peut-être parce que c'était le seul jeu
potable à l'époque.

Ce jeu est tout d'abord l'œuvre de très peu de personnes différentes,
finalement. Il y a moins d'une dizaine de "gros committeurs", leur influence a
donc été primordiale et le parcours du projet peut s'expliquer en analysant
leurs actions. Aujourd'hui, on peut compter environ trois contributeurs sans
parler du Google Summer of Code : les étudiants y participant ne sont jamais
devenus des contributeurs.

Il y a le gars hyper productif qui te pond des features toutes les semaines
sans savoir d'où il les sort. Il a ajouté un support lua sur le serveur, sur le
client, a tout viré, a tout remis, le tout sans explications... Je sais plus
trop où ça en est, a priori ce n'est que sur le serveur, pour une raison
inconnue (probablement pour éviter la triche). Il est pas très gentil sur IRC
mais ça va, il répond aux questions, et quand il voit que de ton côté tu fais
des efforts il va regarder aussi de son côté, sinon il te demandera simplement
si tu ne cherches qu'à lui faire perdre son temps.

Il y a le "chef du projet", qui est arrivé au moment de la libération, code
sous Windows, trolle beaucoup, est super agaçant quand il te répond, c'est le
genre de gars convaincu qu'il sait tout parce qu'il est là depuis longtemps et
fait de l'info depuis longtemps. Sa plus grande qualité c'est d'avoir été
présent durant tout le Summer of Code pour répondre aux questions. Les trois
autres mentors étaient souvent connectés, mais très rarement présents. Le
nombre de fois où j'ai discuté avec lui se compte sur les doigts d'une main. A
part ça, il envoie chier beaucoup de gens, principalement quand on lui demande
quand la prochaine version va sortir. Il a aussi essayé d'implémenter des trucs
pratiques, genre le dead-reckoning pour limiter les effets du lag, sauf que ça
marche pas du tout, mais personne est capable ni d'enlever son code, ni de le
corriger, ce qui est un gros problème. Il est parti sur un projet personnel.

Niveau bugs, on a aussi des paquets qui arrivent dans le bon sens et sont
traités dans le mauvais, et personne ne sait pourquoi, par contre on sait que
ça crée un tas de bugs. Ils se sont mis à paniquer devant le nombre de bugs que
l'an dernier. Mais hey, personne n'a envie de les corriger, donc ils ont fini
par accepter de mettre ce projet en premier pour le GSoC 2010 (aucun autre
projet n'a touché au jeu en lui-même, mais des trucs autour, forcément, c'est
tellement bugué qu'ajouter des features serait trop compliqué et compliquerait
encore les choses).

Je me suis occupé de ce projet, j'ai fait une liste de bugs que je comptais
corriger (pris du bug tracker sur sf.net et du fichier BUGS). J'ai l'impression
d'avoir corrigé les plus importants, mais cela restait au final des bugs
mineurs quand on voit les deux gros bugs que j'ai montré au-dessus qui
empêchent simplement toute forme de release avant qu'ils soient corrigés.
Depuis septembre, il n'y a pas eu un seul commit.

## Comment éviter ?

Au final, les développeurs de BZFlag ont été piégés bêtement, il faut le dire.
Release Early, Release Often a du sens, et si on l'oublie, on se fait vite
avoir. Les deux avantages de ce crédo sont simples :

  * Pour sortir une nouvelle version, elle doit fonctionner, donc on doit
    corriger les bugs que les utilisateurs voient.
  * Cela permet de garder la motivation de tout le monde (même si dans le cas
    de BZFlag, les devs préfèrent parfois développer dans leur coin et pas se
    faire embêter par les utilisateurs - c'est très sérieux, je me moque pas,
    et c'est potentiellement justifié).

C'était aussi évitable en créant des branches pour les fonctionnalités
expérimentales. Cela a d'ailleurs été fait pour une autre fonctionnalité
(server-side shots pour éviter la tricherie), et c'était judicieux, parce que
ça ne marche pas du tout non plus, ça évite d'avoir un énorme bug de plus.

On ne maitrise pas la motivation d'un contributeur dans le libre, il peut très
bien disparaître du jour au lendemain, donc c'est très mal d'accepter du code
qui ne fonctionne pas dans SVN.

## Quel futur ?

BZFlag a besoin d'un ou quelques développeurs qui se motivent pour sauver ce
jeu. On est bien là : sauver le jeu. Il n'y a pas eu de release depuis cinq
ans, malgré quatre années consécutives de contributions via le Summer of Code
dont les ressources ont étés incroyablement utilisées.

Et il ne faut pas compter sur les utilisateurs de la version 2 de BZFlag pour
changer à la version buggée. Beaucoup de joueurs jouent très sérieusement, il y
a de nombreuses ligues et équipes, et ne peuvent pas accepter qu'un coup bien
tiré rate son adversaire, ou que la moitié des informations ne soit pas donnée
sur l'écran d'un des joueurs parce que les paquets ont été mal traités.

Est-il envisageable de forker depuis la version qui marchait ? Oui, mais la
personne qui fera ça se retrouvera probablement seule longtemps, et aura de la
difficulté à rester compatible avec le protocole BZFlag dont la documentation
n'est pas à jour. Si une personne décide de faire ça, elle ferait mieux de se
casser la tête à corriger les deux bugs qui restent, et se faire entendre
suffisamment pour éviter d'autres problèmes de ce genre.

<!-- % vim: set spelllang=fr: -->
