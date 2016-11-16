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

Cela peut servir pour des classes dont l'ordre des attributs/méthodes serait important, telle qu'une énumération (`Enum`).
Cela permet entre autres une meilleure introspection des classes.

Il était autrefois possible d'avoir un dictionnaire ordonné pour les attributs en passant par une métaclasse.
En effet, la méthode `__prepare__` des métaclasses permet de spécialiser la création du dictionnaire `__dict__`, et donc de retourner un `OrderedDict` si besoin.

La volonté de cette PEP (et de la 487) est de réduire le besoin de recourir aux métaclasses pour des problèmes simples.

# Simplification de la personnalisation de classes — PEP 487

## Héritage

Une classe peut maintenant influer sur les classes qui en héritent.
C'est ce que permet la méthode `__init_subclass__` nouvellement ajoutée par cette PEP.
La méthode sera appelée chaque fois qu'une classe fille sera créée (et non instanciée), et la classe fille passée en paramètre.

```python
>>> class SubclassMePlease:
...     def __init_subclass__(cls):
...         print('Creation of', cls)
...
>>> class Ok(SubclassMePlease):
...     pass
...
Creation of <class '__main__.Ok'>
```

Cette méthode de classe reçoit aussi en paramètre l'ensemble des arguments nommés passés lors de l'héritage.
Vous ne le saviez peut-être pas, mais des arguments peuvent être donnés lors d'un héritage (notamment pour la précision de la métaclasse).

```python
>>> class SubclassMePlease:
...     def __init_subclass__(cls, *, name, **kwargs):
...         print('Creation of', cls, 'with name', name)
...         cls.name = name
...
>>> class Roger(SubclassMePlease, name='roger'):
...     pass
...
Creation of <class '__main__.Ok'> with name roger
>>> Roger.name
'roger'
```

## Descripteurs

Cette PEP ajoute aussi une nouvelle méthode spéciale aux descripteurs, la méthode `__set_name__`.
Elle permet au descripteur de savoir quel nom d'attribut lui a été donné au sein de la classe.
En effet, la méthode sera appelée suite à la création de la classe, avec comme arguments la classe et le nom du descripteur.

Cela peut s'avérer utile pour les descripteurs qui donnent accès à un autre attribut de l'objet, dont le nom peut maintenant être interpolé depuis celi du descripteur.

Dans l'exemple suivant, nous avons des descripteurs `width` et `height` qui permettent un accès en lecture à `_width` et `_height`.

```python
class AttributeReader:
    def __set_name__(self, owner, name):
        self.attr = '_{}'.format(name)

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.attr)

class Rect:
    width = AttributeReader()
    height = AttributeReader()

    def __init__(self, width, height):
        self._width, self._height = width, height
```

# Générateurs et compréhensions asynchrones — PEP 525 & 530
