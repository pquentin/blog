Title: S'inspirer de l'apprentissage sémantique humain pour nos pauvres algorithmes
Date: 2012-02-24T18:16:23+01:00
Category: Réactions

Je viens d'écouter une présentation passionante de Josh Tenenbaum
intitulée « Towards More Human-like Machine Learning of Word ». L'idée
est de s'inspirer de l'apprentissage du vocabulaire par les enfants
pour mieux apprendre la sémantique des mots en Traitement Automatique
des Langues.

Points intéressants ici :

  * On utilise le vocabulaire du machine learning pour parler de faits
    étudiés jusque-là en psychologie. Celà montre encore une fois le
    bénéfice tiré des échances entre différents domaines a priori
    différents.
  * En particulier, les deux domaines peuvent profiter l'un de
    l'autre. Le Machine Learning profite des idées d'algorithmes, et
    la psychologie a besoin d'algorithmes d'inférences
    **efficaces**.
  * Finalement, c'est surtout de l'apprentissage non-supervisé ; ce
    qui montre tous les progrès possibles dans le domaine.

Différentes observations prouvent que le cerveau humain fonctionne en
partie avec des statistiques. Nous ne citerons ici que [The Brain as a
Statistical Inference Engine (Eugene Charniak,
2011)](http://aclweb.org/anthology-new/J/J11/J11-4001.pdf), qui est à
la base un discours donné par Charniak au moment de recevoir une
récompense prestigieuse en TAL. De manière générale, il semble que les
"bayésiens" et les gens qui utilisent des modèles graphiques
probabilistes soutiennent cette thèse.

Quoiqu'il en soit, l'objectif de Josh Tenenbaum est ici de montrer en
quoi on peut s'inspirer de l'apprentissage tel qu'il est fait par les
enfants pour développer des algorithmes plus efficaces en
apprentissage automatique. Différents exemples d'apprentissage sont
donnés. Dans de nombreux domaines, on reconnaît que vouloir copier
directement le fonctionnement humain n'est pas possible.

  * dans le jeu vidéo, une [IA doit souvent être mauvaise exprès, et
    pas jouer "comme un
    humain"](http://aigamedev.com/open/article/good-vs-fun/)
  * le but de l'apprentissage automatique n'est pas de raisonner comme
    les humains, mais au contraire de résoudre des problèmes que des
    humains ne pourraient pas forcément résodure
  * les réseaux de neurones sont inspirés du cerveau humain, mais
    restent **extrêmement différents**

Cela étant dit, c'est une source d'information comme une autre, et il
n'y a pas de raison de la négliger. Traitons ici deux exemples donnés
au cours de la présentation :

  * apprentissage des adjectifs ;
  * apprentissage des nombres entiers ;

## Apprentissage des adjectifs

Qu'est-ce que l'adjectif "grand" veut dire ? Son sens dépend fortement
du contexte. En effet, un homme grand et un grand bâtiment sont
probablement deux notions différentes de « grand ». Des expériences
ont étés menées où on a demandé à des gens de dire si telle ou telle
instance d'une classe était grande ou non. Il y a différents modèles
qui semblent correspondre à ces observation (une version qui a bien
marché était de faire un clustering des instances observées, puis de
prendre le cluster le plus "grand"). De manière générale, l'enfant
finit par apprendre une règle pour de tels adjectifs en utilisant un
modèle donné, par exemple « par rapport à des exemples connus
d'arbres, un arbre grand est au moins aussi grand que les 10% d'arbres
les plus grands ».

Une fois qu'un adjectif est appris, il devient beaucoup plus facile de
l'étendre à d'autres adjectifs. Une fois qu'une règle est apprise, on
peut l'appliquer partout. Typiquement, l'enfant est ensuite capable
d'appliquer cette même règle en faisant varier :

  * la classe considérée (ex. arbre, bâtiment, hamburger) ;
  * l'adjectif considéré (ex. grand, bon, joli) ;
  * le pourcentage à atteindre (ex. 10%).

Note : on est très mauvais pour estimer des pourcentages, ici 10% est
simplement un moyen de le représenter en machine. Ça fait partie de
l'effort « appliquer le vocabulaire de l'apprentissage automatique à
un problème jusque-là étudié beaucoup en psychologie.

Par exemple, l'enfant peut ensuite apprendre très vite que bon a un
sens appliqué à la classe « nourriture », la dimension « goût », et un
pourcentage quelconque. Mais qu'il a un autre sens quand appliqué à la
classe « jouet », où la dimension sera plutôt « à quel point il
m'amuse ». On retrouve la notion ultra-classique du sens qui dépend du
contexte.

Ainsi, l'enfant *apprend à apprendre*, ce qui lui permet
ensuite avec beaucoup moins d'exemples concrets de déduire beaucoup de
choses. Il apprend bien plus de mots entre un an et demi et deux ans
qu'il en avait appris jusque-là. Le temps était passé jusque-là à
apprendre à apprendre. Ce n'est donc pas un processus linéaire où on
apprend tant de mots par jours. Cette technique est sûrement une
solution aux manques d'exemples d'entraînement en désambiguïsation
lexicale. Peut-être qu'on les utilise mal ?

## Apprentissage des nombres naturels

Une expérience consistait à demander à un enfant de donner une partie
des ballons présents au sol à un adulte. Si je demande un enfant un,
deux, trois ou quatre ballons, il ne va pas forcément être capable de
m'en donner le nombre exact. En effet, suivant l'âge de l'enfant, il
ne connaît que le concept "un ballon", ou alors il connaît aussi le
concept "deux ballons", etc.  Ainsi, quand on lui demande trois
ballons, il ne sait pas combien il en faut, et n'en donne pas que
trois. À partir de quatre, une procédure récursive rentre en jeu, et
tout d'un coup ça marche pour n ballons.

Pour vérifier cette hypothèse, les chercheurs ont utilisé de
l'inductive programming (avec des primitives comme "un-ballon?",
"deux-ballons?", etc. et ont regardé le programme renvoyé au fur et à
mesure de l'augmentation du nombre d'exemples. Effectivement, leur
programme acquiert, un, deux, trois ballons, puis la fonction
récursive.

## Autres exemples

Avec moins de détails, il explique que les enfants apprennent aussi
des quantifieurs tels que "la plupart", "tous", "aucun". Enfin, des
théories générales permettent de rendre plus tractable la découverte
de lins dans un modèle graphique. En pratique, ça augmente directement
l'efficacité de l'algorithme (en terme du nombre d'exemples
nécessaires). (désolé, c'est pas clair, mais il fallait que ça sorte).

<!-- % vim: set spelllang=fr: -->
