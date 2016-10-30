Title: Problème de l'arrêt : démonstration ?
Date: 2010-01-19T15:29:08+01:00
Category: Algorithmique

## Problème de l'arrêt et décidabilité

Le problème de l'arrêt est un problème classique en décidabilité. Il fait partie des problèmes qui n'ont pas de solution : il est impossible d'écrire un algorithme qui résoud ce problème. La plupart des autres problèmes indécidables reviennent à ce problème. On les montres d'ailleurs par diagonalisation : on montre qu'ils reviennent à résoudre le problème de l'arrêt, et donc qu'ils sont indécidables.

Qu'est-ce qu'un problème décidable ? C'est un problème pour lequel, pour toute entrée, on peut répondre en temps fini au problème par oui ou par non : décider si l'entrée correspond.

Le problème de l'arrêt consiste à prendre en entrée un algorithme, et déterminer si cet algorithme s'arrête ou non (pas de boucle infinie). Sa formalisation se fait dans les machines de Turing, mais le résultat vaut aussi dans un langage qui ne comporterait que des conditions, des boucles, et des affectations, et c'est équivalent à tous les langages que vous utilisez actuellement, nos machines n'étant pas grand chose de plus que de machines de Turing à n bandes.

## Démonstration

Je vais donc programmer en C pour montrer que ce n'est pas possible d'écrire un programme qui résoud le problème de l'arrêt. On suppose que le problème est décidable. Il est donc possible d'écrire une fonction halt(), sans perte de généralité.

    int zero()            int loop()
    {                     {
      return 0;               while (true);
    }                         return 0;
                          }
    int weird()
    {
      if (halt(weird)
        return loop();
      else
        return zero();
    }

 * Si halt(weird) retourne vrai alors weird() retourne loop() (contradiction)</li>
 * Si halt(weird) retourne faux alors weird() retourne zero() (contradiction)</li>

Donc on ne peut pas écrire une fonction halt(), donc le problème de l'arrêt est bien indécidable.

## Théorème de Rice

On peut généraliser ce problème au théorème de Rice qui dit qu'il n'est pas possible de déterminer une propriété non triviale d'un algorithme. Une propriété est une caractéristique d'un algorithme. Exemples : "Il calcule la racine carrée de son entrée", "Il ne s'arrête pas". Elle doit être non triviale : ne pas être vraie pour tous les algorithmes, ou fausse pour tous les algorithmes.

Je reprendrai ici la preuve (la non formelle) présente sur wikipedia. On suppose qu'on a un algorithme qui examine un programme quelconque, et qui sait dire si ce problème calcule bien le carré de son entrée. Voici un exemple de programme de calcul de carré :

    int carre(n) {
      a(i);
      return n*n;
    }

a et i sont une fonction et une entrée quelconque. a peut s'arrêter sur i, ou pas. carre est donc seulement une fonction qui calcule le carré d'une fonction si a ne s'arrête pas. Si on pouvait savoir ça, on saurait résoudre le problème de l'arrêt.

Donc, il est impossible d'obtenir une propriété sur un algorithme. En particulier, on ne peut pas :

  * Savoir si ce programme s'arrête</li>
  * Savoir si c'est un virus (il est donc impossible d'écrire un antivirus infaillible)</li>

## Un peu plus de formalisation ?

Ces résultats peuvent être démontrés formellement en utilisant les machines de Turing universelles. Le principe est d'utiliser un codage sous forme de 0 et de 1 pour coder un algorithme et son entrée. On écrit ensuite une machine de Turing qui sait lire ce codage, et on peut raisonner dessus. 
