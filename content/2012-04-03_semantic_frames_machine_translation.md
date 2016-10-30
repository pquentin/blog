Title: Les frames sémantiques aident à évaluer la traduction automatique
Date: 2012-04-03T10:25:47+02:00
Category: TAL

<p><a href="http://www.cs.ust.hk/~dekai/library/WU_Dekai/LoWu_Ijcai2011.pdf">SMT Versus AI Redux: How Semantic Frames Evaluate MT More Accurately, Chi-kiu Lo and Dekay Wu, 2011 (PDF)</a></p>

<p>Ce papier traite de l'évaluation de la traduction automatique et propose une nouvelle solution : HMEANT. Traditionnellement, l'évaluation est faite par rapport à une ou plusieurs références humaines en utilisant le score BLEU.</p>

<h4>Le score BLEU</h4>

<p>Sans rentrer dans des détails que je maîtrise mal moi-même, le score BLEU compare des suites de mots (n-grams) obtenus dans la traduction qu'on évalue et les traductions de références en utilisant une précision (pourcentage de vrais positifs) modifiée. Selon Wikipédia¹, le score BLEU a longtemps été employé en raison de sa "forte corrélation avec les jugements humains" ; mais ce n'est pas toujours vrai et les systèmes ont tendance à faire du sur-apprentissage. En effet, les systèmes sont conçus uniquement pour obtenir un bon score BLEU et n'ont pas de moyen de savoir si le résultat correspond vraiment à une bonne traduction. Toujours selon Wikipédia, SYSTRAN (un système basé sur des règles qui ne prend pas en compte le score BLEU) aurait été manuellement jugé meilleur que d'autres systèmes statistiques obtenant un meilleur score BLEU.</p>

<p>¹ Les articles Wikipédia liés à la recherche scientifique sont ceux pour lesquels j'ai le moins de confiance a priori, étant donné qu'il est difficile pour un chercheur de rester neutre et de ne pas avantager son "courant de recherche" ou sa "vision des choses". Naturellement, il suffirait de vérifier dans les liens donnés, les papiers concernés, les papiers citant les papiers concernés, etc.</p>

<h4>De meilleures évaluations</h4>

<p>Quoiqu'il en soit, le score BLEU ne prend en compte que des suites de mots et ne donne aucune garantie de validité syntaxique ou sémantique. Il n'est pas pour autant possible d'utiliser une représentation profonde du type logique du premier ordre pour établir si le sens est préservé : il faudrait un humain pour obtenir de tels prédicats logiques, ce qui est contraire à la volonté de garder l'évaluation peu coûteuse. Chi-kiu Lo et Dekai Wu proposent d'utiliser les rôles sémantiques pour évaluer la qualité de la traduction : qui a fait quoi à qui, quand, où, pourquoi et comment ? C'est une représentation sémantique "de surface" qu'il est possible d'acquérir automatiquement (section 7), ou même rapidement par des annotateurs non qualifiés (section 6).</p>

<h4>HMEANT</h4>

<p>Le principe est d'évaluer la similarité entre l'annotation en rôles sémantiques de la traduction initiale par rapport à l'annotation en rôles sémantiques de la traduction. Il s'agit de compter les rôles qu'on retrouve dans la traduction automatique et la correspondance de la traduction de ces rôles (traduction du rôle correcte, incorrecte ou partiellement correcte). Les poids attribués à chaque rôle sont appris automatiquement à partir de scores donnés par des humains. Les poids sont donc choisis pour maximiser la correspondance entre des scores humains et le score HMEANT.</p>

<p>Il faut annoter :</p>
<ul>
	<li>les rôles sémantiques de la référence et de la traduction proposée (possible automatiquement) ;</p>

	<li>la correspondance entre la traduction du texte compris dans ces rôles (fait manuellement dans ce papier).</li>
</ul>

<h4>Remarques</h4>

<p>"Although preserving semantic frames across translations may intuitively sound like a good idea, the history of SMT is littered with surprising failures in attempts to incorporate "obvious" candidates for syntactic models and, more recently, semantic models, so caution is warranted." Un certain nombre de garanties sont données quant à la validité de cette métrique par rapport à ses concurrents.</p>

<p>Il est intéressant d'examiner les poids appris : ils montrent très simplement l'intérêt apporté par chaque rôle (qui, quoi, où, etc.). L'agent (qui) et le thème (quoi) sont les plus importants. Le patient (à qui) n'est pas important, mais c'est probablement du à la difficulté de le différencier avec le thème (quoi) de manière consistante : si les annotateurs humains n'ont pas proposé d'annotation consistante sur ce rôle, il est normal qu'il soit peu improtant.</p>

<p>Ce qui m'a personnellement attiré dans ce papier est l'utilisation de l'annotation en rôles sémantique pour une application concrète et très utilisée. Ce n'est ici que de l'évaluation, mais c'est la porte ouverte à l'utilisation de l'annotation en rôles sémantiques (SRL) pour la traduction automatique (ça a déjà été fait par quelques auteurs). Naturellement, la question d'un sur-apprentissage se pose tout autant qu'avec BLEU, mais il est plus souhaitable de faire un "sur-apprentissage" sémantique que sur des n-grams : on est assuré de préserver la structure du sens.</p>

<p>De telles évaluations plus sémantiques/profondes pour la traduction automatique sont l'objet d'une attention particulière à <a href="http://www.cs.ust.hk/~dekai/ssst/">SSST-6</a> (organisé par les auteurs de ce papier, et d'autres faisant des travaux allant dans le même sens), on peut donc attendre de nouvelles avancées dans ce sens dans les mois qui suivent.</p>
