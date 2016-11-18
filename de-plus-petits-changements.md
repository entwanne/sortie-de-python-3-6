 - [PEP 526](http://www.python.org/dev/peps/pep-0526) : annotations de variables.

Après les paramètres de fonctions, ce sont maintenant toutes les variables qui peuvent être annotées.
Le même principe est conservé : les annotations n'ont aucune utilité dans l'interpréteur Python, mais sont stockées dans l'attribut `__annotations__` du scope parent (classe, module).
Les annotations servent pour les outils externes d'analyse statique du code par exemple (`mypy`).

```python
>>> a : int
>>> b : str = 'hello'
>>> a
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'a' is not defined
>>> b
'hello'
>>> __annotations__
{'a': <class 'int'>, 'b': <class 'str'>}
```

```python
>>> class A:
...     x : int = 0
...     y : int = 0
...     name : str
...
>>> A.__annotations__
{'x': <class 'int'>, 'y': <class 'int'>, 'name': <class 'str'>}
```

 - [PEP 515](http://www.python.org/dev/peps/pep-0515) : *underscores* dans les nombres littéraux.

Les grands nombres compacts ne sont pas toujours aisés à lire. Ainsi, on mettra du temps à déceler l'ordre de grandeur dans `1000000000`.
Des `underscores` peuvent maintenant être utilisés dans l'écriture littérale des nombres, pour le séparer en différents blocs de chiffres : `1_000_000_000`.

```python
>>> 1_000_000 * 1_000
1000000000
>>> 0b_11001100_00000001
52225
```

 - [PEP 506](http://www.python.org/dev/peps/pep-0506) : Ajout d'un module `secrets` pour générer des nombres aléatoires plus sûrs.

 - [PEP 523](http://www.python.org/dev/peps/pep-0523) : Ajout d'une API pour l'évaluation de frames de CPython.

 - [PEP 528](http://www.python.org/dev/peps/pep-0528) et [PEP 529](http://www.python.org/dev/peps/pep-0529) : Utilisation d'UTF-8 pour la console et les chemins de fichiers sur Windows.

 - [PEP 509](http://www.python.org/dev/peps/pep-0509) et [PEP 510](http://www.python.org/dev/peps/pep-0510) : Outils d'optimisation au cœur de l'interpréteur (dictionnaires versionnés et *guards* sur les fonctions).

 - [PEP 495](http://www.python.org/dev/peps/pep-0495) : Différenciation des temps identiques après changement d'heure.

 - [PEP 524](http://www.python.org/dev/peps/pep-0524) : Make os.urandom() blocking on Linux


Cette version 3.6 vient aussi avec son lot de corrections de bogues, que vous pourrez retrouver dans le *changelog* de la version.
