Title: La complexité amortie pour l'apprenti programmeur
Date: 2011-08-22T10:48:49+02:00
Category: Algorithmique

Cet article a pour but d'expliquer un concept de manière simple : le but est que vous ayez une nouvelle flèche dans votre carquois à la fin de cet article. C'est pas extrêmement rigoureux, mais l'important est de retenir le principe, qui est utile au jour le jour. Ceci dit, je serai ravi de corriger les coquilles.

La complexité est un outil très puissant permettant de mesurer l'efficacité d'un algorithme donné. Prenons l'exemple du calcul de la taille d'une liste chaînée, on dit qu'il s'exécute en O(n). Cela signifie que si on double la taille de la liste en entrée, l'algorithme prendra deux fois plus longtemps pour s'exécuter. Au contraire, si la taille est stockée à un endroit particulier, la complexité devient O(1) : on peut s'assurer que quelque soit la taille de la liste, le calcul de la taille prendra le même temps à s'exécuter : on n'a plus à se soucier de la taille de notre liste. La complexité est ainsi d'une importance capitale pour les programmes manipulant de gros volumes de données (par exemple des données venant du web) : un mauvais algorithme pourra prendre des années alors qu'un bon algorithme s'exécutera dans un temps plus raisonnable (par exemples quelques minutes ou heures). C'est un sujet passionant, et je vous recommande « [L'algorithmique pour l'apprenti programmeur](http://zestedesavoir.com/tutoriels/329/algorithmique-pour-lapprenti-programmeur/) » si vous voulez vous y initier. :)

La complexité amortie est un outil algorithmique qui permet, par exemple de passer en O(1) des algorithmes qui seraient normalement en O(n). Le terme « amorti » vient du fait que toutes les n opérations, on fait un investissement qui va s'exécuter en O(n), mais qui sera *amorti* lors des n opérations suivantes qui s'exécuteront elles en O(1). Ce n'est naturellement qu'un exemple et il y a d'autres possibilités que O(n) / O(1) mais c'est le cas le plus courant et aussi celui que je vais illustrer aussi.

Quoi de mieux qu'un exemple pour comprendre de quoi il s'agit ?
---------------------------------------------------------------

Prenons une opération courante en programmation : l'ajout d'un élément à la fin d'un tableau. C'est une opération qui est notamment « encouragée » en C++ : la classe std::vector de la librairie standard C++ comporte une opération push_back().

    for (int i = 0; i &lt; n; i++) {
        v.push_back(i);
    }

Quelle est la complexité de cet algorithme ? Avec une implémentation naïve de push_back(), on obtient du O(n²). En effet, à chaque fois qu'on ajoute un élément, il faut recréer (réallouer) un nouveau tableau avec une case libre supplémentaire : c'est une opération en O(n). Le tout dans une boucle sur n, et paf, O(n²). Pourtant cet algorithme s'exécute en O(n) ! Le principe est simple : à chaque ajout en fin du tableau, au lieu d'agrandir uniquement le tableau d'une case, on double sa taille. Ainsi, la plupart du temps, il suffira de remplir une case inoccupée : opération qui s'exécute en O(1). Comment pourrait-être implémenté push_back() ? Considérons trois membres de la classe `std::vector` :

  * storage, un tableau interne « C » contenant nos éléments
  * capacity, la taille du tableau interne
  * size, la taille du vecteur telle qu'elle est exposée à l'extérieur de la classe

L'implémentation devient :

    void vector::push_back(const T& x) {
        if(size == capacity) {
            reserve(size * 2); // donc capacity = capacity * 2
        }

        storage[size] = x;
        size++;
    }

**Note :** la fonction reserve() permet de réallouer le tableau interne afin de réserver de la place pour une utilisation future : elle prend en paramètre le nombre d'éléments totaux et s'exécute en O(n), n étant l'entier passé en paramètre.

Si jamais la taille (externe) est sur le point de dépasser la capacité (taille du tableau interne), alors on double la capacité. Quoiqu'il arrive, on stocke le nouvel élément à la fin du tableau (c'est-à-dire "size"). Ici, l'appel à reserve() se fait en O(n), et le reste de push_back() s'exécute en O(1). Ainsi, si l'appel à reserve se fait tous les n appels, on a gagné : n opérations tous les n appels revient à une opération à chaque appel. On passe effectivement de O(n) à O(1), étant donné que O(n)/n = 1.

En multipliant par deux la capacité à chaque fois que c'est nécessaire, on ajoute en fait n nouvelles cases vides, prêtes à être remplies. Ainsi, après chaque appel à reserve, les n prochaines opérations se feront en temps constant : c'est effectivement gagné ! L'insertion en fin de tableau s'exécute en O(1) amorti. Il est important de préciser que c'est du O(1) amorti et non du O(1) : cela explicite que certains appels se dérouleront en O(n).

Pour information, la norme C++ impose une complexite constante pour push_back(). Dans le cas de C++0x et de vector, vous pouvez aller voir au paragraphe 23.3.6.5/2 qui dit : « The complexity is linear in the number of elements inserted plus the distance to the end of the vector. ». En d'autres termes pour insérer un seul élément à la fin du vecteur, la complexité est constante. C'est à l'implémentation de choisir la méthode exacte, mais c'est très certainement toujours "on multiplie par un facteur de 2". C'est ce que fait g++, vous pouvez le vérifier. :)

Performance
-----------

J'analyse ici la performance liée à l'implémentation du tableau dynamique. Il ne faut pas généraliser à l'ensemble des algorithmes qui ont une complexité amortie meilleure que la complexité normale. Ainsi, pour l'ajout d'un élément à la fin du tableau, il y a un « coût » : la mémoire utilisée par std::vector sera, dans le pire des cas, le double de la mémoire réellement stockée par le programme. Ce n'est qu'un facteur 2. À titre de comparaison, les listes utilisent deux fois plus de mémoire que les tableaux, donc on peut considérer que c'est négligeable. Ça nous permet de gagner un facteur "n" en temps d'exécution. La classe vector peut aussi s'occuper de réduire la taille du tableau interne si vous supprimez des éléments (via pop_back()). Vous pouvez considérer que c'est trop cher payé, mais ça ne l'est pas, sauf dans des circonstances exceptionnelles. Si vous êtes vraiment inquiets et que vous connaissez à l'avance le nombre d'éléments à insérer, il est possible d'appeller reserve() vous-même avant d'insérer vos éléments. Cependant, c'est une mauvaise idée parce que cela vous apprend à optimiser prématurément quand ce n'est pas utile, ce qui est nuisible. Bjarne Stroustrup (à l'origine de C++) a d'ailleurs [une entrée de sa FAQ C++](http://www.stroustrup.com/bs_faq2.html#slow-containers) sur le sujet :


> People sometimes worry about the cost of std::vector growing incrementally. I used to worry about that and used reserve() to optimize the growth. After measuring my code and repeatedly having trouble finding the performance benefits of reserve() in real programs, I stopped using it except where it is needed to avoid iterator invalidation (a rare case in my code). Again: measure before you optimize.

Ceci n'était qu'une introduction, et « Introduction à l'algorithmique » de Monsieur Cormen (chapitre 17 en l'occurence) vous donnera bien plus d'informations sur le sujet. Cependant, l'apprenti programmeur peut se contenter de cette introduction rapide pour comprendre le concept et ne plus s'inquiéter lors des appels à push_back (ou de l'équivalent dans son langage préféré).
