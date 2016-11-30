# Interpolation de chaînes littérales — [PEP 498](https://www.python.org/dev/peps/pep-0498/)

Loin sont maintenant les `'My name is {name}'.format(name=name)` ou encore `'My name is {}'.format(name)`.
Avec Python 3.6, le formatage de chaînes de caractères gagne en clarté avec l'interpolation littérale.

En plus du préfixe `r` pour définir une chaîne brute, le préfixe `b` pour une chaîne d'octets, arrive maintenant le préfixe `f`, dédié au formatage.
Une chaîne préfixée de `f` sera interpolée à sa création, de manière à résoudre les expressions utilisées au sein de la chaîne de formatage.

La syntaxe pour l'interpolation littérale est reprise sur [la syntaxe de la méthode `str.format`](https://docs.python.org/3/library/string.html#formatspec).
Au détail près que sont accessibles les variables de l'espace de nom courant plutôt que seulement les valeurs qui étaient auparavant passées en arguments.

```python
>>> name = 'John'
>>> f'Hello {name}'
'Hello John'
>>> # Équivalent à
>>> 'Hello {name}'.format(name=name)
'Hello John'
>>> 'Hello {name}'.format(**locals())
'Hello John'
```

Des formatages plus complets sont permis par la méthode `__format__` de certains objets, comme ici pour ceux de type [`datetime.date`](https://docs.python.org/3/library/datetime.html#datetime.date.__format__).

```python
>>> import datetime
>>> date = datetime.date.today()
>>> f'Cet article a été écrit le {date}'
'Cet article a été écrit le 2016-10-27'
>>> f'Cet article a été écrit le {date:%d %B %Y}'
'Cet article a été écrit le 27 octobre 2016'
```

Les accolades peuvent non seulement contenir un nom de variable, mais aussi toute expression Python valide.

```python
>>> nb_pommes = 5
>>> print(f"J'ai {nb_pommes} pomme{'s' if nb_pommes > 1 else ''}.")
J'ai 5 pommes.
>>> nb_pommes = 1
>>> print(f"J'ai {nb_pommes} pomme{'s' if nb_pommes > 1 else ''}.")
J'ai 1 pomme.
```

# Préservation de l'ordre des arguments nommés — [PEP 468](https://www.python.org/dev/peps/pep-0468/)

Les paramètres du type `**kwargs` dans la signature d'une fonction sont maintenant assurés d'être des dictionnaires ordonnés.
Les arguments nommés sont ainsi récupérés dans l'ordre où ils ont été saisis.

```python
def xml_tag(name, **kwargs):
    lines = [f'<{name}>']
    for key, value in kwargs.items():
        lines.append(f'    <{key}>{value}</{key}>')
    lines.append(f'</{name}>')
    return '\n'.join(lines)
```

Cette fonction nous permet de sérialiser un extrait XML tout en conservant le bon ordre des éléments fils.

```python
>>> print(xml_tag('book', title='Notre-Dame de Paris', author='Victor Hugo', year=1831))
<book>
    <title>Notre-Dame de Paris</title>
    <author>Victor Hugo</author>
    <year>1831</year>
</book>
```

Il devient aussi possible de construire facilement un objet `OrderedDict`.

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
| >>> {'z': 0, 'x': 5, 'y': 1}
| {'z': 0, 'x': 5, 'y': 1}
| ```
|
| CPython reprend ici les travaux entrepris par *pypy* pour construire une version des dictionnaires plus compacte, c'est-à-dire occupant moins d'espace en mémoire.

# Protocole de gestion des chemins de fichiers — [PEP 519](https://www.python.org/dev/peps/pep-0519/)

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

Nous avons ici notre propre type d'objet (`UserHome`), qui est interprété par les fonctions systèmes (`os.listdir` dans notre cas) comme un chemin de fichier.

# Préservation de l'ordre des attributs définis dans les classes — [PEP 520](https://www.python.org/dev/peps/pep-0520/)

Similairement à la PEP 468, le dictionnaire des attributs d'une classe est maintenant assuré d'être ordonné.
Il conservera alors l'ordre de définition des attributs et méthodes dans le corps de la classe.

Cela peut servir pour des classes dont l'ordre des attributs/méthodes serait important, telle qu'une énumération (`Enum`).
Cela permet aussi une meilleure introspection des classes.

```python
class AutoEnum:
    red = None
    green = None
    blue = None

i = 0
for name, value in AutoEnum.__dict__.items():
    if not name.startswith('__') and not name.endswith('__') and value is None:
        setattr(AutoEnum, name, i)
        i += 1
```

```python
>>> print(AutoEnum.__dict__)
{'__module__': '__main__', 'red': 0, 'green': 1, 'blue': 2, ...}
```

Il était autrefois possible d'avoir un dictionnaire ordonné pour les attributs en passant par une métaclasse.
En effet, la méthode `__prepare__` des métaclasses permet de spécialiser la création du dictionnaire `__dict__`, et donc de retourner un `OrderedDict` si besoin.

La volonté de cette PEP (et de la 487) est de réduire le besoin de recourir aux métaclasses pour des problèmes simples.

# Simplification de la personnalisation de classes — [PEP 487](https://www.python.org/dev/peps/pep-0487/)

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
Vous ne le saviez peut-être pas, mais des arguments peuvent être donnés lors de la création d'une classe (notamment pour la précision de la métaclasse).
Il était déjà possible auparavant de les récupérer dans les méthodes `__new__` et `__init__` des métaclasses.

```python
>>> class SubclassMePlease:
...     def __init_subclass__(cls, *, name, **kwargs):
...         print('Creation of', cls, 'with name', name)
...         cls.name = name
...
>>> class Roger(SubclassMePlease, name='roger'):
...     pass
...
Creation of <class '__main__.Roger'> with name roger
>>> Roger.name
'roger'
```

## Descripteurs

Cette PEP ajoute aussi une nouvelle méthode spéciale aux descripteurs, la méthode `__set_name__`.
Elle permet au descripteur de savoir quel nom d'attribut lui a été donné au sein de la classe.
En effet, la méthode sera appelée suite à la création de la classe, avec comme arguments la classe et le nom du descripteur.

Cela peut s'avérer utile pour les descripteurs qui donnent accès à un autre attribut de l'objet, dont le nom peut maintenant être interpolé depuis celui du descripteur.

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

# Générateurs et compréhensions asynchrones — [PEP 525](https://www.python.org/dev/peps/pep-0525/) & [PEP 530](https://www.python.org/dev/peps/pep-0530/)

[[a]]
| Les paragraphes qui suivent demandent des connaissance sur le concept de générateur en Python et de programmation asynchrone.

Les mots-clés `async` et `await` [introduits avec Python 3.5](/articles/1568/decouvrons-la-programmation-asynchrone-en-python/#5-la-syntaxe-asynchrone-de-python-3-5) connaissent une nouvelle extension.
En effet, il devient désormais possible de coupler les coroutines avec des générateurs et des compréhensions (listes en intension, *generator expressions*).

C'est-à-dire que l'on va pouvoir définir un générateur asynchrone, qui dépendra d'événements externes pour produire ses valeurs.
L'itération sur ces générateurs devra alors être réalisée *via* `async for` plutôt qu'un simple `for`, au sein de coroutines.

Imaginons un générateur qui produirait les lignes d'un texte en les récupérant depuis un programme distant. Nous le représenterons ici par une itération sur un fichier, faisant une pause d'une seconde après chaque ligne pour illustrer une certaine latence.

```python
import asyncio

async def produce_lines(filename):
    with open(filename, 'r') as f:
        for line in f:
            yield line.rstrip('\n')
            await asyncio.sleep(1)
```

On reconnaît que `produce_lines` est un générateur à l'utilisation du mot-clé `yield` en ligne 6.

Nous pouvons alors avoir une seconde coroutine, `print_lines`, qui itérera sur ce générateur pour afficher les lignes produites.

```python
async def print_lines(filename):
    async for line in produce_lines(filename):
        print(line)
```

Et à l'utilisation :

```python
>>> loop = asyncio.get_event_loop()
>>> loop.run_until_complete(print_lines('corbeau.txt'))
Maître Corbeau, sur un arbre perché,
Tenait en son bec un fromage.
Maître Renard, par l'odeur alléché,
Lui tint à peu près ce langage :
...
```

Quant aux compréhensions asynchrones, introduites par la PEP 530, elles s'illustrent par la coroutine suivante, chargée de retourner la liste de ces lignes.

```python
async def get_lines(filename):
    return [line async for line in produce_lines(filename)]
```

Notez bien le `async for` utilisé dans la compréhension.

L'appel à notre coroutine au sein de notre boucle événementielle nous retourne donc ici la liste des lignes.

```python
>>> loop.run_until_complete(get_lines('corbeau.txt'))
['Maître Corbeau, sur un arbre perché,', 'Tenait en son bec un fromage.', "Maître Renard, par l'odeur alléché,", ...]
```
