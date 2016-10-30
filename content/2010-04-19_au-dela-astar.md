Title: Pathfinding : au-delà d'A* (A star)
Date: 2010-04-19T11:57:50+02:00
Category: Algorithmique

Cet article traitera de différentes techniques utiles si vous avez besoin de faire un peu de pathfinding ou si vous vous demandez comment ça se fait.

## Pré-requis

L'algorithme de base pour faire une recherche d'un point à un autre est A\*. En très gros, le principe est simple, on associe à chaque case un coût, et au lieu d'avoir une file ou une pile comme dans d'autres parcours, on a une liste et on prend l'élément de la liste qui semble être le plus proche de la solution. Il y a plein de tutoriels à ce sujet, vous pouvez [commencer par Wikipédia](http://fr.wikipedia.org/wiki/Algorithme_A*).

## Principe général

Il existe de nombreux algorithmes qui permettent de chercher un chemin entre deux points dans un graphe. L'algorithme le plus efficace est A\*. En fait, il est optimal, mais cela ne règle pas le problème, car
 son efficacité dépend de l'heuristique qu'on lui fournit. Cette heuristique permet à l'algorithme de choisir les points du graphe qui ont le plus de chance de venir à la solution. En général, on utilise la
distance de Manhattan, mais ça peut être n'importe laquelle du moment qu'elle est minorante.

Une heuristique minorante sous-estimera toujours le coût du chemin le plus court, ce qui évitera à A\* de ne pas envisager des sommets du graphe qui pourraient mener à la solution directement. En gros, ça veut dire qu'on peut pas avoir une carte avec possibilité de se téléporter : dans ce cas là, il sera impossible d'avoir une heuristique minorante et donc de faire fonctionner A\* (j'imagine qu'il faudrait alors se rabattre sur [d'autres parcours](http://zestedesavoir.com/tutoriels/329/algorithmique-pour-lapprenti-programmeur/401/quelques-autres-structures-de-donnees-courantes/2026/arbres/)).


## Espace de recherche

On a donc l'algo qui parcourt un graphe, mais comment définir ce graphe ? Il y a différentes solutions suivant le type de jeu ou d'application.

  * Pour un bête jeu tile-based, il suffit que le graphe soit la matrice, avec des coûts donnés pour se déplacer (disons 14 en diagonale, 10 le reste du temps)
  * Si l'espace parcouru est continu, on peut alors le diviser en polygones, les lier entre eux, ce qui nous donne notre graphe
  * En 3D, on peut faire pareil, avec des polygones, et ajouter la composante z à la distance de Manhattan

Ces "polygones" s'appellent dans le jargon des "navigation meshes". Lisez [Fixing pathfinding once and for all](http://www.ai-blog.net/archives/000152.html) (et plus particulièrement l'annexe) si vous voulez l'implémenter. Tous les articles ne sont pas disponibles sur le web, certains réfèrent à l'excellente série des livres "AI Game Programming Wisdom" (seul le Tome 1 est disponible en aperçu sur [Google Books](http://books.google.com/books?id=dHXSvj_M__sC&amp;printsec=frontcover&amp;dq=ai+game+programming+wisdom&amp;hl=fr&amp;cd=1#v=onepage&amp;q&amp;f=false).

Une fois que vous avez vos polygones, il suffit de faire une recherche dans le graphe. Vous avez donc un chemin qui traverse vos polygones. Comment produire un joli chemin pour vos personnages ?

## Funnel Algorithm

C'est là qu'intervient l'algorithme qui en déduit un joli chemin à travers ces polygones. Ça se traduit littéralement par "algorithme de l'entonnoir". Jusqu'à il y a peu de temps il n'y avait pas grand chose sur le net pour faire ça, jusqu'au post [Simple&Stupid Funnal algorithm par Mikko Mononen](http://digestingduck.blogspot.com/2010/03/simple-stupid-funnel-algorithm.html) qui présente une implémentation simple et rapide. Et path, le chemin. :)

## Optimisations haut niveau

Quand le monde est très grand, qu'il y a beaucoup (beaucoup) d'agents et beaucoup (beaucoup) de polygones/tiles, le pathfinding peut devenir un facteur limitant de l'application. Si le monde est particulièrement grand, on peut utiliser du "Level of Detail". C'est principalement utilisé pour le rendu graphique : si ce qu'on veut afficher est loin, on n'affiche qu'une version peu coûteuse, ou on ne l'affiche pas du tout.

Appliqué au pathfinding, cela veut dire qu'on divise le monde en sous-parties, et le coût pour passer de partie en partie est connu. Il suffit donc d'appliquer A\* sur la grande représentation, puis sur chacune des parties pour pouvoir la traverser. On a ensuite le même genre de problèmes qu'avec les polygones : trouver un chemin "lisse". La recherche Google [hierarchical pathfinding](http://www.google.fr/search?hl=en&amp;q=hierarchical+pathfinding) vous donnera un bon aperçu de la littérature sur le sujet.

On peut aussi cacher les requêtes les plus courantes pour ne pas avoir à recalculer. Une autre technique intéressante, qu'il est possible de coupler au pathfinding "hiérarchique" est de d'abord calculer une première estimation du chemin, demander à l'unité de commencer à bouger, puis de rectifier ce chemin par la suite.

## Optimisations bas niveau

"Bas niveau" ici signifie simplement une modification de l'implémentation de l'algorithme, pas de son utilisation. Il y a tout un article à ce sujet dans le premier "AI Game Programming Wisdom" qui décrit tout un nombre de possibilités crades (ils sont notamment tous fiers d'utiliser des templates 90% du temps et d'avoir la possibilité d'utiliser des fonctions virtuelles quand la recherche n'est pas une bête recherche de chemin).

Ils évoquent par contre la notion de "cheap list" qui est intéressante. Dans A\*, on a une liste "ouverte" qui contient la liste des noeuds à explorer. À chaque nouvelle case explorée, on insère dans cette liste les cases adjacentes (huit dans le cas classique), et à chaque nouvelle itération on doit récupérer la case qui est la plus proche du but. Il faut donc une structure de données qui permettre d'insérer rapidement (la liste est adaptée) mais aussi de sélectionner le plus rapidement possible le maximum de la liste.

Maintenir la liste triée aurait été une solution, mais cela impose une insertion en O(n). Du coup, la liste est séparée en deux partie, la première disons de 15 éléments qui est triée, et l'autre qui ne l'est pas. À chaque insertion, si l'élément est plus petit que le minimum de la première liste, on le met dans l'autre, sinon on l'insère (c'est toujours du O(n) mais sur 15 éléments, donc du O(1) :). Si la première liste devient vide, il faut récupérer les quinze premiers éléments de la liste non triée, c'est du O(n), mais comme ça arrive pas souvent, pouf c'est (presque) amorti.

## Eviter des obstacles dynamiques

Trouver un chemin d'un point A à un point B, c'est facile, on sait faire, on sait optimiser, tout ça. Mais comment faire lorsque de nombreux agents se croisent ? Si ils sont pas trop nombreux, on utilise ce qui s'appelle des "steering behaviors". Le principe est simple : si la ligne qu'on est en train de parcourir est bloquée par un obstacle, on tourne un peu pour l'éviter et revenir rapidement dessus en suite.

Comment faire quand ces obstacles deviennent nombreux et dynamiques ? Différentes solutions existent. La <a href="http://grail.cs.washington.edu/projects/crowd-flows/">recherche à ce sujet</a> utilise des solveurs utilisant la "mécanique des fluides". Deux vidéos montrant deux RTS modernes qui arrivent à résoudre ce problème selon leurs vidéos :

  * <a href="http://j.mp/8X2fBA">FlowField Pathfinding in Supreme Commander 2 (GameTrailers.com)</a>
  * <a href="http://j.mp/cvaKSK">350+ Unit Zergling Swarm in StarCraft 2: Pathfinding Demo (YouTube.com)</a>

Un autre sujet de recherche à ce niveau est l'applicaton des [velocity obstacles](http://en.wikipedia.org/wiki/Velocity_obstacle) pour permettre aux personnages de s'éviter. Je vous conseille à nouveau le [blog de Mikko Mokonen](http://digestingduck.blogspot.com/) pour en savoir plus à ce sujet et voir quelques démos.

  * <a href="http://digestingduck.blogspot.com/2010/03/local-navigation-grids.html">Local navigation grids</a>
  * <a href="http://digestingduck.blogspot.com/2010/03/geometric-vs-sampling.html">Geometric vs. Sampling</a>

Il y a toujours possibilité d'améliorer bien sûr, genre en cherchant à tourner de manière réaliste en prenant en compte le personnage (un tank se retourne pas facilement et ne peut pas coller un mur vu sa taille).
