



# Unpacking généralisé – PEP 448

L'*unpacking* est une opération permetant de séparer un itérable en plusieurs
variables :

```python
>>> l = (1, 2, 3, 4)
>>> a, b, *c = l
>>> a
1
>>> b
2
>>> c
(3, 4)
>>> def f(a, b):
...    return a + b, a - b    # f retourne un tuple
...
>>> somme, diff = f(5, 3)
>>> somme
8    # 5 + 3
>>> diff
2    # 5 - 3
```

Il est alors possible de passer un itérable comme argument de fonction comme
si son contenu était passé élément par élément grâce à l'opérateur `*` ou de
passer un dictionnaire comme arguments nommés avec l'opérateur `**` :

```python
>>> def spam(a, b):
...    return a + b
...
>>> l = (1, 2)
>>> spam(*l)
3    # 1 + 2
>>> d = {"b": 2, "a": 3}
>>> spam(**d)
5    # 3 + 2
```

Cette fonctionnalité restait jusqu'à maintenant limitée et les conditions
d'utilisation très strictes. Deux de ces contraintes ont été levées...

## Support de l'unpacking dans les déclarations d'itérables


Lorsque vous souhaitez définir un `tuple`, `list`, `set` ou `dict` littéral, il
est maintenant possible d'utiliser l'*unpacking*.

```python
# Python 3.4
>>> tuple(range(4)) + (4,)
(0, 1, 2, 3, 4)
# Python 3.5
>>> (*range(4), 4)
(0, 1, 2, 3, 4)

# Python 3.4
>>> list(range(4)) + [4]
[0, 1, 2, 3, 4]
# Python 3.5
>>> [*range(4), 4]
[0, 1, 2, 3, 4]

# Python 3.4
>>> set(range(4)) + {4}
set(0, 1, 2, 3, 4)
# Python 3.5
>>> {*range(4), 4}
set(0, 1, 2, 3, 4)

# Python 3.4
>>> d = {'x': 1}
>>> d.update({'y': 2})
>>> d
{'x': 1, 'y': 2}
# Python 3.5
>>> {'x': 1, **{'y': 2}}
{'x': 1, 'y': 2}

# Python 3.4
>>> combinaison = long_dict.copy()
>>> combination.update({'b': 2})
# Python 3.5
>>> combinaison = {**long_dict, 'b': 2}
```

Notamment, il devient maintenant facile de sommer des itérables pour en former
un autre.

```python
>>> l1 = (1, 2)
>>> l2 = [3, 4]
>>> l3 = range(5, 7)

# Python 3.5
>>> combinaison = [*l1, *l2, *l3, 7]
>>> combinaison
[1, 2, 3, 4, 5, 6, 7]

# Python 3.4
>>> combinaison = list(l1) + list(l2) + list(l3) + [7]
>>> combinaison
[1, 2, 3, 4, 5, 6, 7]
```

Cette dernière généralisation permet ainsi d'apporter une symétrie par rapport
aux possibilités précédentes.

```python
# Possible en Python 3.4 et 3.5
>>> elems = [1, 2, 3, 4]
>>> fst, *other, lst = elems
>>> fst
1
>>> other
[2, 3]
>>> lst
4

# Possible uniquement en Python 3.5
>>> fst = 1
>>> other = [2, 3]
>>> lst = 4
>>> elems = fst, *other, lst
>>> elems
(1, 2, 3, 4)
```

## Support de plusieurs unpacking dans les appels de fonctions


Cette généralisation se répercute sur les appels de fonctions ou méthodes :
jusqu'à Python 3.4, un seul itérable pouvait être utilisé lors de l'appel à
une fonction. Cette restriction est maintenant levée.

```python
>>> def spam(a, b, c, d, e):
...    return a + b + c + d + e
...
>>> l1 = (2, 1)
>>> l2 = (4, 5)
>>> spam(*l1, 3, *l2)        # Légal en Python 3.5, impossible en Python 3.4
15                           # 2 + 1 + 3 + 4 + 5
>>> d1 = {"b": 2}
>>> d2 = {"d": 1, "a": 5, "e": 4}
>>> spam(**d1, c=3, **d2)    # Légal en Python 3.5, impossible en Python 3.4
15                           # 5 + 2 + 3 + 1 + 4
>>> spam(*(1, 2), **{"c": 3, "e": 4, "d": 5})
15                           # 1 + 2 + 3 + 5 + 4
```

Notez que si un nom de paramètre est présent dans plusieurs dictionnaires,
c'est la valeur du dernier qui sera prise en compte.

D'autres généralisations [sont mentionnées dans la PEP](https://www.python.org/dev/peps/pep-0448/#variations)
mais n'ont pas été implémentées à cause d'un manque de popularité parmi les
developpeurs de Python. Il n'est cependant pas impossible que d'autres
fonctionnalités soient introduites dans les prochaines versions.

# Opérateur de multiplication matricielle – PEP 465


L'introduction d'un opérateur binaire n'est pas courant dans Python.
Aucun dans la série 3.x, le dernier ajout semble être
[l'opérateur `//` dans Python 2.2](https://www.python.org/dev/peps/pep-0238/)[^ndbp_op_bin].
Regardons donc pour quelles raisons celui-ci a été introduit.


[^ndbp_op_bin]: Ces ajouts sont tellement peu courants qu'il est difficile de
trouver des traces de ces modifications. L'opérateur `//` semble être le seul
ajout de toute la série 2.

## Signification



Ce nouvel opérateur est dédié à la multiplication matricielle. En effet,
comme tous ceux ayant fait un peu de mathématiques algébriques le savent sûrement, 
on définit généralement la multiplication matricielle par l'opération suivante 
(pour des matrices $2*2$ ici) :

$$
\begin{pmatrix}a&b \\ c&d \end{pmatrix} \cdot \begin{pmatrix}e&f \\ g&h \end{pmatrix} =
\begin{pmatrix}a*e+b*g&a*f+b*h \\ c*e+d*g&c*f+d*h \end{pmatrix}
$$

Or il est souvent aussi nécessaire d'effectuer des multiplications
terme à terme :

$$
\begin{pmatrix}a&b \\ c&d \end{pmatrix} * \begin{pmatrix}e&f \\ g&h \end{pmatrix} =
\begin{pmatrix}a*e&b*f \\ c*e&d*h \end{pmatrix}
$$

Tandis que certains langages spécialisés possèdent des opérateurs dédiés pour
chacune de ces opérations[^ndbp_op_matmatlab] il n'y a en Python rien de
similaire. Avec la bibliothèque [numpy](http://www.numpy.org), la plus
populaire pour le calcul numérique dans l'éco-système Python, il est possible
d'utiliser l'opérateur natif `*` pour effectuer une multiplication terme à
terme, surcharge d'opérateur se rencontrant dans la plupart des bibliothèques
faisant intervenir les matrices. Mais il n'existe ainsi plus de moyen simple
pour effectuer une multiplication matricielle.
Avec `numpy`, il est pour le moment nécessaire d'utiliser la méthode `dot` et
d'écrire des lignes de la forme :

```python
S = (A.dot(B) - C).T.dot(inv(A.dot(D).dot(A.T))).dot(A.dot(B) - C)
```

Celle-ci traduit cette formule :

$$
(A \times B - C)^T \times (A \times D \times A^T)^{-1} \times (A \times B - C)
$$

Cela n'aide pas à la lecture... Le nouvel opérateur `@` est introduit et
dédié à la multiplication matricielle. Il permettra d'obtenir des expressions
équivalentes de la forme :

```python
S = (A @ B - C).T @ inv(A @ D @ A.T) @ (A @ B - C)
```

Un peu mieux, non ?

[^ndbp_op_matmatlab]: Par exemple, avec Matlab et Julia, `*` / `.*` servent respectivement pour la multiplication matricielle et terme à terme.

## Impact sur vos codes



Cette introduction devrait être anodine pour beaucoup d'utilisateurs. En effet,
aucun objet de la bibliothèque standard ne va l'utiliser[^ndbp_nused]. Cet
opérateur binaire servira principalement à des bibliothèques annexes, à
commencer par *numpy*.

[^ndbp_nused]: Ce qui est rare mais existe déjà. En particulier l'objet `Elipsis` créé avec `...`.

Si vous souhaitez supporter cet opérateur, trois méthodes spéciales peuvent
être implémentées : `__matmul__` et `__rmatmul__` pour la forme `a @ b` et
`__imatmul__` pour la forme `a @= b`, de façon similaire aux autres opérateurs binaires.

À noter qu'il est déconseillé d'utiliser cet opérateur pour autre chose que
les multiplications matricielles.

## Motivation



L'introduction de cet opérateur est un modèle du genre : il peut sembler très
spécifique mais est pleinement justifié. La lecture de la PEP, développée, est
très instructive. 

Pour la faire adopter, les principales bibliothèques
scientifiques en Python ont préparé cette PEP ensemble pour arriver à une
solution convenant à la grande partie de la communauté scientifique, assurant
dès lors l'adoption rapide de cet opérateur. La PEP précise ainsi l’intérêt et
les problèmes engendrés par les autres solutions utilisées jusque-là. 

Enfin
cette PEP a été fortement appuyée par [la grande popularité de Python dans le monde scientifique](https://www.python.org/dev/peps/pep-0465/#but-isn-t-matrix-multiplication-a-pretty-niche-requirement).
On apprend aussi que `numpy` est le module n’appartenant pas à la bibliothèque
standard le plus utilisé parmi tous les codes Python présents sur Github et ce
sans compter d'autres bibliothèques comme `pylab` ou `scipy` qui vont aussi
profiter de cette modification et comptent parmi les bibliothèques les plus
communes.

# Support des coroutines – PEP 492


### Introduction à la programmation asynchrone



Pour comprendre ce qu'est la programmation asynchrone, penchons-nous sur l'exemple
de code suivant, chargé de télécharger le contenu de la page d’accueil de
*Zeste de Savoir* et de l'enregistrer sur le disque par morceaux de 512 octets
en utilisant la bibliothèque [requests](http://www.python-requests.org/en/latest/).

```python
import requests

def fetch_page(url, filename, chunk_size=512):

    # Nous effectuons une demande de connexion au serveur HTTP et attendons sa réponse
    response = requests.get(url, stream=True)

    # Nous vérifions qu'il n'y a pas eu d'erreur lors de la connexion
    assert response.status_code == 200

    # Nous ouvrons le fichier d'écriture
    with open(filename, mode='wb') as fd:

        # Nous récupérons le fichier de la page d'accueil par morceaux : à chaque
        # tour de boucle, nous demandons un paquet réseau au serveur et attendons
        # qu'il nous le donne
        for chunk in response.iter_content(chunk_size=chunk_size):

            # Nous écrivons dans le fichier de sortie, sur le disque
            fd.write(chunk)

if __name__ == "__main__":
    fetch_page('http://zestedesavoir.com/', 'out.html')
```

Lors d'un appel à ce genre de fonction très simple votre processeur passe son temps... à ne
rien faire ! En effet, quand il n'attend pas le serveur Web, il patiente le temps
que le disque effectue les opérations d'écriture demandées. Or tout cela est très
lent à l'échelle d'un processeur, et tous ces appels à des services extérieurs
sont très courants en Web : connexion et requêtes à une base de données, appels à
une API externe, etc. La perte de temps est de ce fait considérable.

La programmation dite **asynchrone** cherche à résoudre ce problème en permettant
au développeur d'indiquer les points où la fonction attend un service extérieur
(base de données, serveur Web, disque dur, etc.), appelé *entrées/sorties* (en
anglais : *io*). Le programme principal peut ainsi faire autre chose pendant ce
temps, comme traiter la requête d'un autre client. Pour cela une boucle
évenementielle est utilisée. Le coeur de l'application est alors une fonction
qu'on pourrait résumer par les opérations suivantes :

 1. Prendre une tâche disponible
 2. Exécuter la tâche jusqu'à ce qu'elle soit terminée ou qu'elle doive attendre des entrées/sorties
 3. Dans le second cas, la mettre dans une liste de tâches en attente
 4. Mettre dans la liste des tâches disponibles les tâches en attente ayant reçu leurs données
 5. Retourner en 1

Prenons un exemple, avec un serveur Web :

- Un client `A` demande une page, appelant ainsi une fonction `f`
- Un client `B` demande une page, appelant une autre fonction `g`, appel mis en attente vu que le processeur est occupé avec le client `A`
- L'appel à `f` atteint une écriture dans un fichier, et se met en pause le temps que le disque fasse son travail
- `g` est démarrée
- L'écriture sur le disque de `f` est terminée
- `g` atteint une demande de connexion à la base de données et se met en pause le temps que la base réagisse
- L'appel à `f` est poursuivi et terminé
- Un client `C` demande une page
- etc.

En découpant une fonction par morceaux, la boucle peut exécuter du code pendant
que les longues opérations d'entrées/sorties d'un autre morceau de code se
déroulent à l'extérieur...

[[i]]
| Nous nous comportons naturellement de cette manière dans la vie. Par exemple,
| lorsque vous effectuez un rapport que vous devez faire relire à votre chef : après
| lui avoir fait parvenir une première version, vous allez devoir attendre qu'il
| l'ait étudié avant de le corriger ; plutôt que de patienter bêtement devant son
| bureau, vous vaquez à vos occupations, et serez informé lorsque le rapport pourra
| être repris.

### La programmation asynchrone en Python



La programmation asynchrone a été remise récemment au goût du jour par
[Node.js](https://nodejs.org/). Mais Python n'est pas en reste, avec de multiples
bibliothèques pour gérer ce paradigme : [Twisted](https://twistedmatrix.com/trac/)
existe depuis 13 ans (2002), [Tornado](http://www.tornadoweb.org/en/stable/) a
été libérée par [FriendFeed](http://blog.friendfeed.com/) il y a 6 ans (2009),
à l'époque où le développement de [gevent](http://www.gevent.org/) commençait.

Au moment de la version 3.4 de Python, [Guido van Rossum](https://fr.wikipedia.org/wiki/Guido_van_Rossum),
créateur et BDFL du langage, a décidé de standardiser cette approche
en synthétisant les idées des bibliothèques populaires précédemment citées et
d'intégrer une implémentation typique dans la bibliothèque standard : c'est le
module [asyncio](https://docs.python.org/3/library/asyncio.html). Ce dernier n'a
pas pour but de remplacer les bibliothèques mentionnées, mais de proposer
une implémentation standardisée d'une boucle événementielle et de mettre à
disposition quelques fonctionnalités bas niveau (*Tornado* peut ainsi
[être utilisé avec asyncio](http://tornado.readthedocs.org/en/latest/asyncio.html)).

*[BDFL]: *Benevolent Dictator For Life* (dictateur bienveillant à vie)

Tandis que *Node.js*, par exemple, emploie ce que l'on appelle des fonctions de
rappel (*callbacks*) pour ordonnancer les différentes étapes d'un algorithme,
*asyncio* utilise des coroutines, lesquelles permettent d'écrire des fonctions
asynchrones avec un style procédural. Les
[coroutines](https://fr.wikipedia.org/wiki/Coroutine) ressemblent beaucoup aux
fonctions, à la différence près que leur exécution peut être suspendue et reprise
à plusieurs endroits dans la fonction. Python dispose déjà de constructions de
ce genre : les générateurs[^ndbp_gen]. C'est ainsi avec les générateurs que
*asyncio* a été initialement développé.

[^ndbp_gen]: Les générateurs sont en théorie
[des formes particulières de coroutines](https://en.wikipedia.org/wiki/Coroutine#Comparison_with_generators),
mais leur implémentation en Python leur confère pleinement le statut de coroutine.

Pour illustrer cela, reprenons l'exemple décrit plus haut, avec cette fois Python 3.4,
*asyncio*, la bibliothèque [aiohttp](https://github.com/KeepSafe/aiohttp)[^ndbp_aiohttp]
pour faire des requêtes HTTP et la bibliothèque [aiofiles](https://github.com/Tinche/aiofiles/)
pour écrire dans des fichiers locaux, le tout de façon asynchrone :

[^ndbp_aiohttp]: Bibliothèque qui propose des fonctions d'entrées/sorties sur le protocole HTTP.

```python
import asyncio
import aiohttp
import aiofiles

@asyncio.coroutine           # On déclare cette fonction comme étant une coroutine
def fetch_page(url, filename, chunk_size=512):
    # À la ligne suivante, la fonction est interrompue tant que la requête n'est pas revenue
    # Le programme peut donc vaquer à d'autres tâches
    response = yield from aiohttp.request('GET', url)

    # Le serveur HTTP nous a répondu, on reprend l'appel de la fonction à ce
    # niveau et vérifie que la connexion est correctement établie
    assert response.status == 200

    # De même, on suspend ici la fonction le temps que le fichier soit créé
    fd = yield from aiofiles.open(filename, mode='wb')
    try:
        while True:
            # On demande un paquet au serveur Web et interrompt la coroutine le
            # temps qu'il nous réponde
            chunk = yield from response.content.read(chunk_size)

            if not chunk:
                break

            # On suspend la coroutine le temps que le disque dur écrive dans le fichier
            yield from fd.write(chunk)

    finally:
        # On ferme le fichier de manière asynchrone
        yield from fd.close()

if __name__ == "__main__":
    # On démarre une tâche dans la boucle évènementielle.
    # Vu qu'il n'y en a qu'une seule, procéder de manière asynchrone n'est pas très utile ici.
    asyncio.get_event_loop().run_until_complete(fetch_page('http://zestedesavoir.com/', 'out.html'))
```

Cet exemple nous montre comment utiliser les générateurs et *asyncio* pour obtenir
un code asynchrone lisible. Toutefois :

 - Il y a détournement du rôle d'origine de l'instruction `yield from`. Sans le décorateur la fonction pourrait être facilement confondue avec un générateur classique, sans rapport avec de l'asynchrone.
 - Tandis que l'exemple d'origine utilisait `with` pour assurer la fermeture du fichier même en cas d'exception, nous sommes obligés ici de reproduire manuellement le comportement de cette instruction. En effet `with` n'est pas prévue pour appeler des coroutines, donc on ne peut effectuer de manière asynchrone les opérations d'entrée (ligne 16) ni de sortie (ligne 31).
 - De la même façon, l'instruction `for` ne permet pas de lire un itérable de manière asynchrone, comme fait ligne 21. Il nous faut donc passer peu élégamment par une boucle `while`.

## Les nouveaux mot-clés



Python 3.5 introduit deux nouveaux mot-clés pour résoudre les problèmes précédemment
cités : `async` et `await`. De l'extérieur, `async` vient remplacer le décorateur
`asyncio.coroutine` et `await` l'expression `yield from`.

Mais les modifications sont plus importantes que ces simples synonymes. Tout
d'abord, une coroutine déclarée avec `async` n'est **pas** un générateur. Même si en
interne les deux types de fonctions partagent une grande partie de leur implémentation,
il s'agit de constructions du langage différentes et il est possible que les différences
se creusent dans les prochaines versions de Python. Pour marquer cette distinction
et le fait que des générateurs sont une forme restreinte de coroutines, il est
possible dans Python 3.5 d'utiliser des générateurs partout où une coroutine est
attendue, mais pas l'inverse. Le module *asyncio* continue ainsi à supporter les
deux formes.

L'ajout de ces mot-clés a aussi été l'occasion d'introduire la possibilité d'itérer
de manière asynchrone sur des objets. Ce support, assuré en interne par les nouvelles
méthodes `__aiter__` et `__anext__` pourra être utilisé par des bibliothèques
comme *aiohttp* pour simplifier les itérations grâce à la nouvelle instruction `async for`.

De la même façon, des *context managers* asynchrones font leur apparition via
l'instruction `async with`, en utilisant les méthodes `__aenter__` et `__aexit__`.

Il est donc possible, et conseillé, en Python 3.5 de ré-écrire le code précédent de la manière
suivante.

```python
import asyncio
import aiohttp
import aiofiles

# On déclare cette fonction comme étant une coroutine
async def fetch_page(url, filename, chunk_size=512):
    # À la ligne suivante, la fonction est interrompue tant que la connexion n'est pas établie
    response = await aiohttp.request('GET', url)
    assert response.status == 200

    # Ouverture, puis fermeture, du fichier de sortie de manière asynchrone
    async with aiofiles.open(filename, mode='wb') as fd:

        # On lit le contenu au fur et à mesure que les paquets arrivent et on interrompt l'appel en attendant
        async for chunk in response.content.read_chunk(chunk_size):

            await fd.write(chunk)

if __name__ == "__main__":
    asyncio.get_event_loop().run_until_complete(fetch_page('http://zestedesavoir.com/', 'out.html'))
```

L'exemple ci-dessus montre clairement l'objectif de l'ajout de `async for` et `async with` : si les `async`
et `await` sont ignorés, la coroutine est fortement similaire à une implémentation
utilisant des fonctions classiques présentées en début de section.

[[a]]
| Le dernier exemple de codes est un aperçu de ce que pourrait être la
| programmation asynchrone avec Python 3.5. À l'heure où ces lignes sont écrites, 
| *aiohttp* et *aiohttp* ne supportent pas encore les nouvelles instructions `async for`
| et `async with`. De la même manière, pour l'instant, seul *asyncio* gère les mot-clés
| `async` et `await` au niveau de sa boucle événementielle.

Enfin, notez que les expressions `await`, au-delà de leur précédence beaucoup plus
faible, sont moins restreintes que les `yield from` et peuvent être placées partout
où une expression est attendue. Ainsi les codes suivants sont valides.

```python
if await foo:
    pass

# /!\ Différent de `async with`
# Ici, c'est `bar` qui est appelé de manière asynchrone et non `bar.__enter__`
with await bar:
     pass

while await spam:
     pass

await spam + await egg
```

## Impact sur vos codes



Concernant vos codes existants, cette version introduit logiquement la dépréciation
de l'utilisation de `async` et `await` comme noms de variable. Pour le moment, un
simple avertissement sera émis, lequel sera transformé en erreur dans Python 3.7.
Ces modifications restent donc parfaitement rétro-compatibles avec Python 3.4.

Si vous utilisez *asyncio*, rien ne change pour vous et vous pouvez continuer à
employer les générateurs si vous souhaitez conserver le support de Python 3.4.
L'ajout des méthodes de la forme `__a****__` n'est donc pas obligatoire, mais peut 
vous permettre cependant de supporter les nouveautés de Python 3.5.

# Annotations de types – PEP 484


Les annotations de fonctions [existent depuis Python 3](https://www.python.org/dev/peps/pep-3107/)
et permettent d'attacher des objets Python aux arguments
et à la valeur de retour d'une fonction ou d'une méthode :

```python
def ma_fonction(param: str, param2: 42) -> ["Mon", "annotation"]:
    pass
```

Depuis le début, il est possible d'utiliser n'importe quel objet valide comme
annotation. Ce qui peut surprendre, c'est que l'interpréteur n'en fait pas
d'utilisation particulière : il se contente de les stocker dans l'attribut
`__annotations__` de la fonction correspondante :

```python
>>> ma_fonction.__annotations__
{'param2': 42, 'return': ['Mon', 'annotation'], 'param': <class 'str'>}
```

La PEP 484 introduite avec Python 3.5 fixe des conventions pour ces annotations :
elles deviennent réservées à « l'allusion de types » (*Type Hinting*), qui
consiste à indiquer le type des arguments et de la valeur de retour des
fonctions :

```python
# Cette fonction prend en argument une chaîne de caractères et en retourne une autre.
def bonjour(nom: str) -> str:
    return 'Zestueusement ' + nom
```

Toutefois, il ne s'agit encore que de conventions : l'interpréteur Python ne
fera rien d'autre que de les stocker dans l'attribut `__annotations__`. Il ne
sera même pas gêné par une annotation d'une autre forme qu'un type.

Pour des indications plus complètes sur les types, un module `typing` est
introduit. Il permet de définir des types génériques, des tableaux, etc. Il serait
trop long de détailler ici toutes les possibilités de ce module, donc nous nous
contentons de l'exemple suivant.

```python
# On importe depuis le module typing :
# - TypeVar pour définir un ensemble de types possibles
# - Iterable pour décrire un élément itérable
# - Tuple pour décrire un tuple
from typing import TypeVar, Iterable, Tuple

# Nous définissons ici un type générique pouvant être un des types numériques cités.
T = TypeVar('T', int, float, complex)
# Vecteur représente un itérateur (list, tuple, etc.) contenant des tuples 
# comportant chacun deux éléments de type T.
Vecteur = Iterable[Tuple[T, T]]

# Pour déclarer un itérable de tuples, sans spécifier le contenu de ces derniers, 
# nous pouvons utiliser le type natif :
# Vecteur2 = Iterable[tuple]
# Les éléments présents dans le module typing sont là pour permettre une 
# description plus complète des types de base.

# Nous définissons une fonction prenant un vecteur en argument et renvoyant un nombre
def inproduct(v: Vecteur) -> T:
    return sum(x*y for x, y in v)

vec = [(1, 2), (3, 4), (5, 6)]
res = inproduct(vec)
# res == 1 * 2 + 3 * 4 + 5 * 6 == 44
```

Néanmoins, les annotations peuvent surcharger les déclarations et les rendre
peu lisibles. Cette PEP a donc introduit une convention supplémentaire : les
fichiers `Stub`. Il a été convenu que de tels fichiers, facultatifs, contiendraient
les annotations de types, permettant ainsi de profiter de ces informations sans
polluer le code source. Par exemple, le module `datetime` de la bibliothèque 
standard comporterait un fichier `Stub` de la forme suivante.


```python
class date(object):
    def __init__(self, year: int, month: int, day: int): ...
    @classmethod
    def fromtimestamp(cls, timestamp: int or float) -> date: ...
    @classmethod
    def fromordinal(cls, ordinal: int) -> date: ...
    @classmethod
    def today(self) -> date: ...
    def ctime(self) -> str: ...
    def weekday(self) -> int: ...
```

Ces fichiers permettent aussi d'annoter les fonctions de bibliothèques écrites 
en C ou de fournir des annotations de types aux modules utilisant déjà les 
annotations pour autre chose que le *type hinting*. 

Tout comme rien ne vous oblige à utiliser les annotations pour le typage, rien
ne vous force à vous servir du module `typing` ou des fichiers `Stub` :
l'interpréteur n'utilisera ni ne vérifiera toujours pas ces annotations ; le
contenu des fichiers `Stub` ne sera même pas intégré à l'attribut `__annotations__`. Ne
vous attendez donc pas à une augmentation de performances en utilisant les
annotations de types : l'objectif de cette PEP est de normaliser ces informations
pour permettre à des outils externes de les utiliser. Les premiers bénéficiaires
seront donc les EDI, les générateurs de documentation (comme [Sphinx](http://sphinx-doc.org/))
ou encore les analyseurs de code statiques comme [mypy](http://mypy-lang.org/),
lequel a fortement inspiré cette PEP et sera probablement le premier outil à
proposer un support complet de cette fonctionnalité.

Les annotations de types ne sont donc qu'un ensemble de conventions, comme il en
existe déjà plusieurs dans le monde Python ([PEP 333](https://www.python.org/dev/peps/pep-0333/),
[PEP 8](https://www.python.org/dev/peps/pep-0008/), [PEP 257](https://www.python.org/dev/peps/pep-0257/),
etc.). Cet ajout n'a donc aucun impact direct sur vos codes mais permettra aux
outils externes de vous fournir du support supplémentaire.


