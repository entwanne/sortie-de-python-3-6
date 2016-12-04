Personne ne les décidant par avance, les fonctionnalités de la future version 3.7 ne sont pas encore connues.
Néanmoins, ces fonctionnalités découlent des propositions faites par les utilisateurs/développeurs, généralement sur l'une de ces deux *mailing lists* :

 - [*python-dev*](https://mail.python.org/mailman/listinfo/python-dev) pour les changements concernant l'intepréteur CPython ;
 - [*python-ideas*](https://mail.python.org/mailman/listinfo/python-ideas) pour les idées générales sur le langage.

Ces idées sont ensuite débattues, et mènent à une [PEP](https://www.python.org/dev/peps/) si elles sont acceptées par une majorité de développeurs.
La PEP sera elle-même acceptée ou refusée 
(par Guido, le BDFL, ou par un autre développeur si Guido est l'auteur de la PEP). En
l'absence de feuilles de route définies à l'avance, les PEP peuvent arriver
tard dans le cycle de développement.

*[BDFL]: Benevolent Dictator for Life (Dictateur bienveillant à vie)
*[PEP]: Python Enhancement Proposal (Proposition d'amélioration de Python)

[[i]]
| Partant de ces propositions, nous pouvons établir une liste de possibles futures fonctionnalités.

Nous ne reparlerons pas ici des **propriétés de classes** et des **sous-interpréteurs**, abordés dans [le précédent article](/articles/175/sortie-de-python-3-5/#4-ce-que-lon-peut-attendre-pour-la-version-3-6-1).

# Syntaxe pour les arguments uniquement positionnels ([PEP 457](https://www.python.org/dev/peps/pep-0457/))

Les arguments positionnels en Python sont les arguments passés aux fonctions qui sont assignés aux paramètres suivant leur position.
Par exemple, avec une fonction `def add(x, y): pass`, dans l'expression `add(3, 5)`, le paramètre `x` récupérera la valeur du premier argument (3), et `y` celle du second (5).
Avec une expression telle que `add(x=3, y=5)` (ou `add(y=5, x=3)` équivalente), la correspondance se fait par le nom des paramètres, on parle alors d'arguments nommés.

Les paramètres d'une fonction en Python peuvent alors correspondre à des arguments positionnels ou des arguments nommés,
il est aussi possible de faire en sorte qu'ils ne puissent être définis que *via* des arguments nommés (avec une fonction telle que `def add(*, x, y)` par exemple).

Néanmoins, bien que prévus par l'implémentation (la fonction `pow` ne reçoit que des arguments positionnels), il n'existe aucune syntaxe en Python permettant d'avoir des arguments uniquement positionnels.
Il est ici proposé de pouvoir ajouter un caractère `/` lors de la définition des paramètres, lequel serait précédé des arguments uniquement positionnels.

```python
def add(x, y, /):
    return x + y
```

# Expressions attrapant les exceptions ([PEP 463](https://www.python.org/dev/peps/pep-0463/))

Cette PEP vise à permettre d'attraper des exceptions sous forme d'expressions, évitant de créer un bloc `try`/`except` dans les cas les plus simples.

L'idée serait d'avoir une expression `expression except exception_type: default_value` qui évaluerait l'expression `expression` et prendrait sa valeur, mais dans le cas où `expression` lèverait une exception de type `exception_type`, le résultat de l'expression serait `default_value`.

Ainsi, les deux codes suivants seraient équivalents.

```python
result = a / b except ZeroDivisionError: 0
```

```python
try:
    result = a / b
except ZeroDivisionError:
    result = 0
```

# Généralisation de l'interpolation de chaînes ([PEP 501](https://www.python.org/dev/peps/pep-0501/))

L'interpolation de chaînes arrivée avec Python 3.6 ouvre de nouvelles perspectives pour les versions suivantes.

Avec le préfixe `f`, les chaînes sont automatiquement formatées à l'exécution, à partir de la méthode `__format__` des objets référencés. Il n'est donc possible ni d'appliquer plusieurs fois le formatage, ni de personnaliser ce dernier en fonction du contexte, ce qui peut être intéressant pour des raisons de sécurité notamment (on souhaitera par exemple formater de façon particulière une chaîne lorsqu'elle est intégrée à une requête SQL, pour contrer les [injections](https://zestedesavoir.com/articles/193/les-injections-sql/)).

Cette PEP souhaite pallier ces manques en introduisant le préfixe `i`. L'expression `i'...'` retournerait un objet d'un nouveau type (un *template*) dont la méthode `__format__` (appelée par `format(...)`) se chargerait du formatage proprement dit. En particulier, `format(i'Bonjour {name}')` serait équivalent à `f'Bonjour {name}'`. Il deviendrait aussi possible d'étendre le type de *template* pour implémenter sa propre méthode `__format__` et bénéficier d'un formatage différent.
 
# Opérateurs de coalescence ([PEP 505](https://www.python.org/dev/peps/pep-0505/))

Les opérateurs de coalescence (*null-coalescing*) sont des opérateurs qui simplifient la gestion des valeurs nulles, que l'on retrouve dans plusieurs langages (C#, Ruby, Perl, PHP, etc.).
Ils permettent de n'exécuter la suite d'une expression que si la valeur utilisée n'est pas nulle.

Le concept pourrait être adapté en Python autour de la valeur `None`.
Il est courant de rencontrer des expressions telles que `obj.attr if obj is not None else 'default'` pour récupérer l'attribut `attr` d'un objet `obj` incertain (pouvant être un objet du type désiré ou `None`).
Cette expression présente le problème de s'avérer un peu lourde et d'être répétitive (`obj` y apparaît deux fois).
Si `obj` était une expression plus complète, on pourrait aussi avoir un soucis de double-évaluation.

Les opérateurs de coalescence seraient au nombre de 3 :

 -  `??` (*None-coalescing*), afin d'avoir une valeur par défaut si l'expression à gauche de l'opérateur s'évalue à `None` ;
 -  `?.` (*None-aware attribute access*), pour n'accéder à l'attribut d'un objet que si cet objet n'est pas `None` (comme dans l'exemple plus haut) ;
 -  `?.[]` (*None-aware indexing*) permet de faire de même lors de l'accès aux éléments d'un conteneur (`container[...]`).

On aurait alors :

```python
>>> number = 5
>>> number ?? 100
5
>>> number = None
>>> number ?? 100
100
>>> number = 0
>>> number ?? 100
0

>>> obj = {'a': 5}
>>> obj?.popitem('a')
5
>>> obj = None
>>> obj?.popitem('a')
None

>>> obj = {'a': 5}
>>> obj?.['a']
5
>>> obj = None
>>> obj?.['a']
None
```
