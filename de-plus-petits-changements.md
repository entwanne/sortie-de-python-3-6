
 - [PEP 441](http://www.python.org/dev/peps/pep-0441) : ajout d'un module `zipapp`, pour améliorer le support et la création d'applications packagées sous forme d'archive `zip`, introduits dans Python 2.6.
 - [PEP 461](https://www.python.org/dev/peps/pep-0461/) : retour de l'opérateur de formatage `%` pour les types de données `byte` et `bytearray`, de la même façon qu'il était utilisable pour les types chaînes (`str`) dans Python 2.7.
 - [PEP 488](https://www.python.org/dev/peps/pep-0488/) : la suppression des fichiers `pyo`.
 - [PEP 489](https://www.python.org/dev/peps/pep-0489/) : un changement de la procédure d'import des modules externes.
 - [PEP 471](http://www.python.org/dev/peps/pep-0471) : une nouvelle fonction `os.scandir()`, pour itérer plus efficacement sur le contenu d'un dossier sur le disque qu'en utilisant `os.walk()`. Cette dernière fonction l'utilise maintenant en interne et est de trois à cinq fois plus rapide sur les systèmes POSIX et de sept à vingt fois plus rapide sur Windows.
 - [PEP 475](http://www.python.org/dev/peps/pep-0475) : l'interpréteur va maintenant automatiquement retenter le dernier appel système lorsqu'un signal `EINTR` est reçu, évitant aux codes utilisateurs de s'en occuper.
 - [PEP 479](https://www.python.org/dev/peps/pep-0479/) : le comportement des générateurs changent lorsqu'une exception `StopIteration` est déclenchée à l'intérieur. Avec cette PEP, cette exception est transformée en `RuntimeError` plutôt que de quitter silencieusement le générateur. Cette modification cherche à faciliter le déboguage et clarifie la façon de quitter un générateur : utiliser `return` plutôt que de générer une exception. Cette PEP n'étant pas rétro-compatible, vous devez manuellement l'activer avec `from __future__ import generator_stop` pour en profiter.
 - [PEP 485](https://www.python.org/dev/peps/pep-0485/) : une fonction `isclose` est ajoutée pour tester la « proximité » de deux nombres flottants.
 - [PEP 486](https://www.python.org/dev/peps/pep-0486/) : le support de `virtualenv` dans l'installateur de Python sous Windows est amélioré et simplifié.
 - [Ticket 9951](https://bugs.python.org/issue9951) : pour les types `byte` et `bytearray`, une nouvelle méthode `hex()` permet maintenant de récupérer une chaîne représentant le contenu sous forme hexadécimale.
 - [Ticket 16991](https://bugs.python.org/issue16991) : l'implémentation en C, plutôt qu'en Python, des `OrderedDict` (dictionnaires ordonnés) apporte des gains de quatre à cent sur les performances.

De plus, comme d’habitude, tout un tas de fonctions, de petites modifications 
et de corrections de bugs ont été apportées à la bibliothèque standard, 
qu'il serait trop long de citer entièrement.

Notons aussi que le support de Windows XP est supprimé. Cela ne veut pas dire que 
Python 3.5 ne fonctionnera pas dessus, mais rien ne sera fait par les développeurs 
pour cela.

Enfin, le développement de CPython 3.5 (à partir de la première *release* candidate) 
s'est effectué sur [BitBucket](https://bitbucket.org/larry/cpython350) plutôt que 
sur le traditionnel [dépôt auto-hébergé](https://hg.python.org/cpython) de la 
fondation, afin de bénéficier des fonctionnalités proposées par le site ; le 
gestionnaire de versions est toujours Mercurial. Il n'est pas encore connu si 
les prochaines versions suivront ce changement et si le développement de CPython 
se déplacera sur BitBucket : il s'agit d'une expérimentation que les développeurs 
vont évaluer. Pour le moment, les branches pour les versions 3.5.1 et 3.6 restent 
sur [https://hg.python.org/cpython](https://hg.python.org/cpython) au moins jusqu'à 
la sortie de Python 3.5.


