Title: Les métaphores de navigation choisies par Gnome
Date: 2010-01-06T01:39:00+01:00
Category: Logiciel

Si vous avez déjà utilisé un "Vanilla" Gnome, vous aviez peut-être
remarqué que Nautilus ouvre une nouvelle fenêtre à chaque fois que
vous voulez ouvrir un nouveau dossier. C'est quelque chose de
complètement horrible à utiliser mais cependant justifié d'un point de
vue ergonomique.

C'est en fait la bataille entre deux "métaphores" utilisés pour de
telles GUIs. La première, la métaphore orienté objet, consiste à
considérer chaque fichier, chaque dossier, chaque élément comme un
objet au sens de la POO, avec ses caractéristiques et des informations
telles que sa position sur l'écran, sa taille, etc. stockés entre les
différents affichages de cet élément. Si vous ouvrez un article que
vous aviez fermé il y a deux semaines, il se retrouvera au même
endroit, avec la même taille, à la même page, etc.

L'inconvénient de cette méthode exercée à l'abus c'est que vous vous
retrouverez avec un nombre incalculable de fenêtres dès que vous
voulez fouiller un peu dans vos dossiers.

Récemment, nautilus a renoncé à activer cette fonctionnalité par
défaut. La raison principale, c'est le Gnome Shell. C'est la nouvelle
façon de penser le bureau pour Gnome 3.0. C'est d'ailleurs assez
intéressant, je vous invite à lire [la page dédiée au Gnome
Shell](http://live.gnome.org/GnomeShell), et plus particulièrement le
design document si ça vous intéresse. Ça propose un moyen plus simple
d'accéder à ses fenêtres qui sont regroupées par activité, ça
centralise les événements (réception d'un nouveau message, demande
d'une application) pour les gérer au niveau du bureau. Si vous êtes
occupés, vous pouvez filtrer une partie des messages, et les
applications pourront limiter le nombre de popups qu'elles vous
jettent à la figure. Le tout saupoudré de pas mal de recherche sur
l'ergonomie, ça donne plutôt envie d'essayer.

Du coup donc, la principale manière d'accéder à ses fichiers ne sera
plus Nautilus mais un autre machin, ce qui a permi à Nautilus de
revenir à la métaphore de navigation, où on ne crée notamment pas de
nouvelle fenêtre. L'autre raison, c'est que personne n'utilisait ça de
toute façon. :)

Plus d'infos sur
[http://www.bytebot.net/geekdocs/spatial-nautilus.html](http://www.bytebot.net/geekdocs/spatial-nautilus.html)
qui lie vers tous les liens qui vont bien. Have fun.

<!-- % vim: set spelllang=fr: -->
