 - [PEP 526](http://www.python.org/dev/peps/pep-0526) : annotations de variables.

Après les paramètres de fonctions, ce sont maintenant toutes les variables qui peuvent être annotées.
Le même principe est conservé : les annotations n'ont aucune utilité dans l'interpréteur Python, mais sont stockées dans l'attribut `__annotations__` du scope parent (fonction, classe, module).
Les annotations servent pour les outils externes d'analyse statique du code par exemple (`mypy`).

 - [PEP 515](http://www.python.org/dev/peps/pep-0515) : *underscores* dans les nombres littéraux.

Les grands nombres compacts ne sont pas toujours aisés à lire. Ainsi, on mettra du temps à déceler l'ordre de grandeur dans `1000000000`.
Des `underscores` peuvent maintenant être utilisés dans l'écriture littérale des nombres, pour le séparer en différents blocs de chiffres : `1_000_000_000`.


Cette version 3.6 vient aussi avec son lot de corrections de bogues, dont l'énumération serait inutile et fastidieuse.
