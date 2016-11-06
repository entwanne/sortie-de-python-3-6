# Interpolation de chaînes littérales — PEP 498

Loin sont maintenant les `'My name is {name}'.format(name=name)` ou encore `'My name is {}'.format(name)`.
Avec Python 3.6, le formatage de chaînes de caractères gagne en clarté avec l'interpolation littérale.

En plus du préfixe `r` pour définir une chaîne brute, le préfixe `b` pour une chaîne d'octets, arrive maintenant le préfixe `f`, dédié au formatage.
Une chaîne préfixée de `f` sera interpolée à sa création, de manière à résoudre les expressions utilisées au sein de la chaîne de formatage.

```python
>>> import datetime
>>> date = datetime.date.today()
>>> f'Cet article a été écrit le {date}'
'Cet article a été écrit le 2016-10-27'
>>> f'Cet article a été écrit le {date:%d %B %Y}'
'Cet article a été écrit le 27 octobre 2016'
```

(ces exemples de formattage sont permis par [la méthode `__format__` des objets `datetime.date`](https://docs.python.org/3/library/datetime.html#datetime.date.__format__).

Les accolades peuvent non seulement contenir un nom de variable, mais aussi toute expression Python valide.

```python
>>> nb_pommes = 5
>>> print(f"J'ai {nb_pommes} pomme{'s' if nb_pommes > 1 else ''}.")
J'ai 5 pommes.
>>> nb_pommes = 1
>>> print(f"J'ai {nb_pommes} pomme{'s' if nb_pommes > 1 else ''}.")
J'ai 1 pomme.
```

# Préservation de l'ordre des arguments nommés — PEP 468

Les paramètres du type `**kwargs` dans la signature d'une fonction sont maintenant assurés d'être des dictionnaires ordonnés.
Ils sont ainsi récupérés dans l'ordre où ils ont été saisis.

Cela permet par exemple de construire facilement un objet `OrderedDict`.

```python
from collections import OrderedDict
coords = OrderedDict(x=0, y=5, z=1)
```

Là où une liste de tuples était nécessaire dans les versions antérieures de Python.

```python
coords = OrderedDict([('x', 0), ('y', 5), ('z', 1)])
```

[[i]]
| On notera que cela passe, dans CPython 3.6, par une implémentation ordonnée de tous les `dict`.
|
| ```python
| >>> {'x': 0, 'y': 5, 'z': 1}
| {'x': 0, 'y': 5, 'z': 1}
| ```

# Protocole de gestion des chemins de fichiers — PEP 519

Cette PEP revient sur la `pathlib`, bibliothèque de gestion des chemins de fichiers, pour lui ajouter son propre protocole.

Spécialisée dans la manipulation de chemins de fichiers, cette bibliothèque comporte aussi de nombreux outils pour opérer sur ces fichiers (création de fichier, de dossier, suppression, etc.)

Une nouvelle méthode spéciale est maintenant disponible pour nos objets : la méthode `__fspath__`. Elle ne prend aucun paramètre et doit retourner une chaîne de caractères.
Tout objet implémentant cette méthode est considéré par Python comme un chemin, et peut alors être utilisé avec les fonctions Python gérant des chemins (`open`, `os.path.join`, etc.).

```python
import os

class UserHome:
    def __init__(self, username):
        self.username = username

    def __fspath__(self):
        return f'/home/{self.username}'

for filename in os.listdir(UserHome('clem')):
    print(filename)
```

# Préservation de l'ordre des attributs définis dans les classes — PEP 520

Similairement à la PEP 468, le dictionnaire des attributs d'une classe est maintenant assuré d'être ordonné.
Il conservera alors l'ordre de définition des attributs dans le corps de la classe.

Cela permet entre autres une meilleure introspection des classes.

# Simplification de la personnalisation de classes — PEP 487

# Générateurs asynchrones — PEP 525
