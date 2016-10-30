Title: Du web en Java ?
Date: 2010-01-08T19:33:41+01:00
Category: Logiciel

Ce billet est là pour raconter rapidement comment on peut faire du web
en Java. Je ne recommande ça à personne, c'est simplement que j'aurais
été curieux de savoir comment ça marchait avant. C'est quelque chose
qui est apparemment pas mal utilisé en entreprise tout ça.

L'avantage principal que ces entreprises doivent justement y voir,
c'est le fait qu'à chaque livrable, il suffit de donner au client un
simple fichier (un .war) qui contient tout le code du site, les
librairies utiles, etc. Le client n'a plus qu'à déployer le fichier
.war chez lui, et paf, tout marche comme par magie, donc il est
content. Le fournisseur n'a pas besoin de montrer les sources ni de
s'occuper de l'installation, donc il est content. Et voilà, tout le
monde est content, et on obtient un truc qui est pas mal utilisé.

Le client et le développeur n'ont qu'à avoir un conteneur Java EE,
"Tomcat" par exemple, qui s'occupe des détails. Dans la pratique, un
.war c'est simplement des dossiers agencés selon une organisation
particulière, et le tout zippé.

Au niveau de code, de base quand une requête est envoyée, le conteneur
la redirige vers le bon projet puis vers la bonne "servlet".
Concrètement une servlet c'est un controlleur : c'est là que sont
envoyées les requêtes, il faut [implémenter doGet(request, response)
et doPost(request,
response](http://java.sun.com/javaee/5/docs/api/javax/servlet/http/HttpServlet.html)
pour jouer avec. On peut ensuite coder en Java pur, en écrivant la
réponse directement depuis le Java, ou alors en passant par une vue.

Au fur et à mesure des versions (comme d'habitude en Java) ont été
greffés par dessus le tout des nouveaux trucs pour que des [non
codeurs puissent mettre en place des
boucles](http://www.oracle.com/technetwork/java/javaee/jsp/index.html)
(un langage de template particulièrement verbeux en XML), puis des
nouveaux frameworks sont nés, Spring, Roo, tous plus [géniaux les uns
que les autres](https://www.youtube.com/watch?v=Gb1Z0lfl52I) géniaux
les uns que les autres.</a>

Aucun intérêt à en faire tout seul ceci dit.

<!-- % vim: set spelllang=fr: -->
