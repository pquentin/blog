Title: Extraire les connexions les plus fortes d'un graphe
Date: 2012-01-18T16:02:49+01:00
Category: Algorithmique
status: hidden

## Besoins

Il n'est pas rare d'avoir des graphes très connectés et de vouloir en extraire les informations les plus pertinentes. C'est en réalité une forme de clustering.

 * J'en ai eu besoin quand j'avais créé un graphe de relations entre personnes d'un salon IRC, une relation pouvant être le fait de parler en même temps, voire quelque chose de plus fort : un hilight. Ce sont effectivement les deux indices que j'utilise quand je veux savoir qui parle à qui, et se priver d'une de ces informations m'empêcherait d'extraire les clusters voulus.
 * Aujourd'hui, je retrouve ce problème dans l'induction automatique de sens de mots. On veut regrouper les utilisations d'un même mot qui partagent le même sens.

## Algorithme : Shared Nearest Neighbors

### Clustering : voisins communs

On commence par filtrer les arcs les moins intéressants dans un nouveau graphe. Deux éléments sont liés dans le nouveau graphe si ils partagent au moins un voisin commun dans le graphe de départ (on peut naturellement ajuster le nombre de voisins communs suivant les besoins, et sauter cette étape peut parfois produire de meilleurs résultats).

Une fois que ceci est fait, on obtient les clusters voulus.

### Choix de la tête de cluster

Pour donner un nom au cluster, ou tout simplement pour obtenir un "noeud de départ" à partir duquel nous pourrons réaliser nos opérations, on choisit le noeud qui contient le plus de liaisons avec les autres du même cluster.

## Un exemple

<p style="width: 60%; margin: 0 auto"><img style="max-width: 100%; box-sizing: border-box" src="{filename}/images/clustering-chat.png" alt="Clustering des sens du mot chat" /><br /> Source : Ressources et méthodes semi-supervisées pour l'analyse sémantique du texte en français - Claire Mouton</p>
