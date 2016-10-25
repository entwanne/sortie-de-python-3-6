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


Cette version 3.6 vient aussi avec son lot de corrections de bogues, que vous pourrez retrouver dans le *changelog* de la version.
