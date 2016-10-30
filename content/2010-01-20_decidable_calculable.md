Title: Décidable, oui, mais calculable ?
Date: 2010-01-20T18:44:06+01:00
Category: Algorithmique

Après une explication rapide sur les enjeux de la décidabilité, j'ai jugé
intéressant de parler de quelques classes de complexité. La plupart d'entre
vous a sûrement déjà entendu parler de "NP-complet", mais tout le monde ne sait
pas forcément ce que c'est. Ce qu'il faut savoir :

## Classes de complexité

  * Les problèmes dits simples ont des solutions qui s'exécutent en temps
    polynomial : ils appartiennent à la classe P. Cette classe regroupe la plus
    grande partie des problèmes traités aujourd'hui par l'informatique.
    Exemples :
    * Un produit de matrice se fait en O(n^x), x compris entre 2 et 3.
    * Un tri rapide s'exécute en O(n log(n)).
    * Les algorithmes les plus efficaces en terme de traduction se font en
      O(n^5) (à base de Tree-adjoining grammar).
  * Il existe des problèmes dit "exponentiels" : le meilleur algorithme
    existant possible est exponentiel. Rien à faire, dès que l'entrée dépasse
    une taille de 10, il n'est pas possible d'avoir un algorithme praticable en
    raison de l'explosion combinatoire. Exemple : IA d'un jeu d'échec, Go, etc.

## Une classe particulière : la classe NP

C'est entre ces deux classes de problèmes que se situe la classe NP. Les
problèmes de cette classe ont tous la même caractéristique : il est possible de
vérifier une solution donnée à un problème de la classe NP en temps polynomial.
Par exemple, on peut vérifier facilement que le résultat d'un tri est correct :
on regarde si notre tableau est trié (O(n)), donc P est inclus dans NP. Il
existe cependant des problèmes qui sont dans NP mais dont on n'a pas prouvé
l'appartenance à P. Ces problèmes sont les problèmes NP-complets.

## Problèmes NP-complets

Les problèmes NP-complets sont concrètement les problèmes les plus durs de la
classe NP, et sont tous équivalents. Il est possible de démontrer que le
problème SAT est NP-complet (je sais pas faire), et de montrer son équivalence
à d'autres problèmes (plus facile). Par exemple, 3-SAT est équivalent. On peut
de la même manière réduire le problème 3-SAT à un problème de coloration de
graphe, etc.

Une rapide liste de problèmes équivalents :

  * génération d'emploi du temps ;
  * problème du voyageur de commerce ;
  * problème du sac à dos ;
  * problème de la clique (applicable aux réseaux sociaux) ...

L'astuce pour savoir si un problème donné est NP-complet est assez simple : si
la seule possibilité c'est de tester toutes les possibilités, et de recenser
les possibilités correctes, le problème est certainement NP-complet. (Il est
assez facile de savoir si une possibilité est valide grâce à la propriété
principale de la classe NP : elle se fait en temps polynomial.) On peut ensuite
chercher une complexité plus précise, par exemple, générer un emploi du temps
consiste à placer k cours dans n créneaux. La solution connue la plus efficace
est exponentielle, mais on peut vérifier une solution en temps exponentiel :
c'est ce qui fait du problème un problème NP-complet.

## P = NP ?

Le problème « P = NP » peut être considéré comme le problème le plus important
qui reste non résolu en informatique. On sait que P est inclus dans NP, mais on
ne sait pas si NP est inclus dans la classe des problèmes exponentiels ou dans
la classe des problèmes polynomiaux. Millenium propose un million de dollars à
la personne qui saura résoudre ce problème. :)

Pour montrer P = NP, il suffit de montrer que les problèmes NP-complets sont
polynomiaux : ce sont les plus durs de NP, donc si ils sont P, NP = P. Pour
montrer que les problèmes NP-complets sont dans P, il suffit de montrer qu'un
seul de ces problèmes a une solution polynomiale, étant donné qu'ils sont tous
équivalents. Personne n'a réussi jusque là. :)

## Complexité en temps, en espace ?

Le dernier point que vous pouvez vouloir retenir est que la complexité peut
s'exprimer de deux manières : en temps, et en espace. Je n'ai parlé ici que de
la complexité en temps, pour une raison simple : un problème qui a une
complexité donnée en temps ne pourra pas avoir une complexité pire en espace.
Intuitivement, dans un temps donné, on ne peut pas toucher à une infinité de
cases mémoires.
