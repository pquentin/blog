Title: Évaluation de systèmes complexes : quelques pistes
Date: 2012-01-20T14:45:53+01:00
Category: TAL

<p>L'évaluation des systèmes complexes est depuis toujours une partie centrale de l'activité des développeurs travaillant dessus. Le sujet est extrêmement vaste ; regardez la page <a href="http://en.wikipedia.org/wiki/Software_testing">Software testing</a> de la Wikipédia anglaise pour vous en rendre compte. Ce n'est pas non plus pour rien que Google recrute des <a href="http://www.google.com/intl/en/jobs/uslocations/mountain-view/swe/software-engineer-in-test-mountain-view/index.html">Software Engineers in Test</a>.</p>

<p>Quand le système est complexe mais spécifiable, il « suffit » de lister les différents cas possibles et de vérifier que le système a le bon comportement. Cela ne fonctionne pas pour les projets les plus intéressants ; les projets pour lesquels on ne sait pas quelle est la bonne réponse.</p>
<ul>
<li>Comment évaluer un système de reconnaissance de caractères ?</li>
<li>Comment est-ce qu'un moteur de recherche peut dire que sa recherche fonctionne et s'assurer qu'il n'a pas oublié <strong>le</strong> document qui aurait répondu à la question de l'utilisateur ?</li>
<li>Comment est-ce qu'on peut dire si une modification sur une IA de Go a amélioré l'efficacité du système ?</li>
<li>Comment évaluer un système complexe formé comme une « pipeline » de sous-modules ?</li>
<li>Comment évaluer un système de désambiguïsation du sens des mots ?</li>
</ul>

<h4>Le cas de l'apprentissage supervisé</h4>

<p>La première question est la plus facile parce qu'un humain peut répondre à la question. Que pensez-vous faire quand vous utilisez reCAPTCHA pour compléter un formulaire ? Vous participez de manière active à l'amélioration d'un système de reconnaissance de texte qui utilise ces données (l'image et le texte correspondant) pour entraîner les algorithmes de classification. Pour savoir si le système est efficace, il faut prendre des images annotées par des utilisateurs, et vérifier que le système obtient le même résultat.</p>

<p>On obtient alors un unique nombre réel qui est facile à utiliser : si ce nombre augmente lors de la prochaine version du logiciel, on a certainement amélioré la situation, sinon il faut revoir sa copie.</p>

<h4>Précision et rappel</h4>

<p>Pour la deuxième question, la réponse est qu'on peut évaluer la précision, mais pas le rappel. C'est-à-dire qu'on peut savoir quelle proportion des documents retournés sont pertinents, mais pas savoir combien de documents on a raté (surtout quand il y a autant de documents que de pages webs). C'est bien sûr plus complexe que ça ; il y a différents degrés de pertinence, et on peut vouloir adapter la réponse à l'utilisateur. Un autre problème : les requêtes sont exprimées en langage naturel, et sont donc naturellement ambigües. Quand je fais une recherche sur « l'école de voile du port », Google me renvoie des résultats sur le port du voile à l'école.</p>

<h4>Comparaison relative</h4>

<p>La réponse à la troisième question est simple : on ne peut pas évaluer la performance d'une IA sans adversaire, il faut donc lui en donner, et regardant comment est-ce que notre IA se classe parmi ces adversaires, que ce soit d'autres IAs ou des joueurs humains. On utilise aussi souvent des « baselines » qui sont des systèmes représentant une « stratégie simple voire bête » que les IAs doivent savoir contrer. En chifoumi, c'est le « tout aléatoire ». Pour d'autres jeux, ça pourrait être « attaque frontale ». En Traitement Automatique des Langues et à Prologin, on est souvent surpris de l'efficacité des stratégies les plus simples.</p>

<h4>Sous modules</h4>

<p>Quatrième question. Les systèmes de traitement linguistique sont souvent des chaînes, qui réalisent les étapes les unes à la suite des autres. On peut commencer par découper un texte en phrase, puis les phrases en mots, puis de chercher les mots dans un dictionnaire, avant de déterminer quelle est la nature de chaque mot (porte est-il un adjectif ou un adverbe ?) pour ensuite déterminer la structure syntaxique de la phrase avant de déterminer le sens de cette même phrase.</p>

<p>Tous ces modules peuvent être évalués indépendamment, ce qui simplifie beaucoup la tâche. Quand c'est supervisé, on sait faire, quand ça ne l'est pas, on avise (pour la tâche de la désambiguïsation du sens des mots, voir la question suivante).</p>

<p><strong>Note :</strong> une fois que ces modules sont évalués, il existe un moyen efficace de savoir sur quel module travailler. On fait la supposition qu'on pourrait choisir un des modules et de le rendre parfait (100% de bons résultats). Lequel des modules faudrait-il alors choisir ? Il faut regarder l'impact sur la précision finale. Si nous avons quatre modules, quelle est l'amélioration finale en remplaçant le premier module par un résultat parfait ? Et si on remplaçait le second ? Le troisième ? Le quatrième et dernier ? Celui pour lequel le gain est le plus important représente le module à améliorer : il a plus de potentiel que tous les autres pour améliorer le système entier. Ce n'est pas nécessairement le modèle le moins effiace dans l'absolu !</p>

<h4>Technique du bananaphone</h4>

<p>L'évaluation est aussi centrale en « Word Sense Disambiguation », tâche qui consiste à choisir le sens le plus probable parmi tous les <a href="http://duckduckgo.com/?q=!wordnet+organ">sens d'un mot</a>. Une technique consiste à considérer toutes les occurrences de deux mots, par exemple de banana et phone, puis de les concaténer. Exemple de transformation :</p>
<blockquote><p>I talked about my banana over the phone.</p></blockquote>
<p>devient</p>
<blockquote><p>I talked about my bananaphone over the bananaphone.</p></blockquote>

<p>On donne la deuxième phrase à notre système, et on examine quel sens a été choisi à chaque double mot. Si le système retourne un des sens de « banana » pour le premier mot et un des sens de « phone » pour le second, alors il a réussi. Il faut naturellement faire attention à utiliser des mots qui ont des sens bien distincts et qui ne se répètent pas trop souvent (on regarde les contextes gauche et droit pour identifier le sens, si le contexte inclut notre construction « bananaphone », c'est bof). Des gens ont vraiment travaillé sur une utilisation efficace des pseudo-mots (<a href="http://acl.ldc.upenn.edu/N/N03/N03-2023.pdf">Nakov &amp; Hearst, 2003</a>).</p>
