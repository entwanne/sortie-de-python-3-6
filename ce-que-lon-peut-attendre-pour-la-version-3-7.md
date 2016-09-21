Il est impossible de savoir précisément ce qui sera disponible dans la prochaine version du langage et de l'interpréteur.
En effet aucune entreprise ou groupe ne décide par avance des fonctionnalités. Les changements inclus dépendent des
propositions faites, majoritairement, sur deux *mailling list* :

 - [*python-dev*](https://mail.python.org/mailman/listinfo/python-dev) quand cela concerne des changements internes à l'intepréteur CPython.
 - [*python-ideas*](https://mail.python.org/mailman/listinfo/python-ideas) quand cela concerne des idées générales à propos du langage.

S'ensuit une série de débats qui, si les idées sont acceptées par une
majorité de developpeurs principaux, conduit à la rédaction d'une
[PEP](https://www.python.org/dev/peps/), qui sera elle-même acceptée ou refusée 
(par Guido, le BDFL, ou par un autre développeur si Guido est l'auteur de la PEP). En
l'absence de feuilles de route définies à l'avance, les PEP peuvent arriver
tard dans le cycle de développement. Ainsi, la PEP 492 sur `async` et `await`
n'a été créé qu'un mois avant la sortie de la première bêta de Python 3.5.

[[a]]
| Malgré ces incertitudes, on peut deviner quelques modifications probables. Attention, cette section reste très spéculative...

# Continuité des changements introduits dans Python 3.5



Trois thématiques de modifications amorcées dans Python 3.4 et surtout Python 3.5 pourraient être de nouveau sources d'ajouts
importants dans Python 3.6 :

 - La programmation asynchrone avec *asyncio* et les coroutines
 - Les indications de types
 - La généralisation de l'*unpacking*

Les deux premiers éléments sont officiellement en attente de retours de la part des utilisateurs de Python.
Les développeurs de l'interpréteur attendent de connaître les problèmes et limitations rencontrées en utilisant ces
fonctionnalités pour les peaufiner dans les prochaines versions. Python 3.6 devrait donc logiquement voir des améliorations
dans ces deux sections en profitant des retours. Déjà quelques remarques ont été faites, comme
[la complexité de mêler des fonctions synchrones et asynchrones](http://code.activestate.com/lists/python-ideas/34419/).

La généralisation de l'*unpacking* pourrait elle aussi continuer. La
[PEP 448](https://www.python.org/dev/peps/pep-0448/#variations) proposait initialement d'autres généralisations qui n'ont
pas été retenues pour Python 3.5 par manque de temps. Elles seront donc propablement rapidement rediscutées. De plus, ces ajouts dans Python 3.5 ont
donné des idées à d'autres développeurs et quelques [nouvelles modifications](http://code.activestate.com/lists/python-ideas/35074/)
ont été déjà proposées.


# Conservation de l'ordre des arguments fournis à **kwargs lors de l'appel aux fonctions


Cette [PEP](http://www.python.org/dev/peps/pep-0468) a déjà été acceptée mais n'a pas pu être implémentée dans la version 3.5.
Il est donc possible qu'elle soit finalement présente dans la prochaine version. L'idée est que les fonctions puissent connaître
l'ordre dans lequel les arguments nommés ont été passés. Prenons un exemple :

```python
def spam(**kwargs):
    for k, v in kwargs.items():
        print(k, v)

spam(a=1, b=2)
```

Avec Python 3.5, comme toutes versions de CPython et presque toutes les autres implémentations (sauf PyPy), nous ne pouvons
pas savoir si le résultat sera :

Nom | Valeur
----|------
a   |  1
b   |  2

ou

Nom | Valeur
----|------
a   |  2
b   |  1

En effet, un dictionnaire est utilisé pour passer les arguments. Or ceux-ci ne garantissent pas d'ordre sur leurs clés. Pourtant, 
Python dispose d'une classe [`OrderedDict`](https://docs.python.org/3/library/collections.html#collections.OrderedDict) depuis
Python 3.1. L'implémentation de cette PEP se résumerait donc à remplacer l'objet utilisé en interne, un dictionnaire basique, par 
un dictionnaire ordonné. Cependant, jusqu'à maintenant, cet objet était défini en Python. L'implémentation était donc loin
d'être la plus efficace possible et ne pouvait pas être utilisée pour un élément aussi critique du langage. C'est pour cette
raison que la classe a été réimplémentée en C pour Python 3.5, comme noté dans la section « De plus petits changements ».
Maintenant, plus rien ne semble bloquer l'implémentation de cette PEP, qui devrait donc voir le jour dans Python 3.6.

Le principal intérêt de cette modification est que le fonctionnement actuel est contre-intuitif pour bon nombre d'utilisateurs
connaissant mal le fonctionnement interne de Python. D'autres raisons sont invoquées dans la PEP :

 - L'initialisation d'un `OrderedDict` est un cas d'appel où l'ordre des paramètres importe, et de tels objets pourraient maintenant être créés en utilisant des arguments nommés, comme le permettent les dictionnaires.
 - La sérialisation : dans certains formats l'ordre d'apparition des données a de l'importance (ex : l'ordre des colonnes
   dans un fichier CSV). Cette nouvelle possibilité permettrait de les définir plus facilement, en même temps que des
   valeurs par défaut. Elle permettrait aussi à des formats comme XML, JSON ou Yaml de garantir l'ordre d'apparition
   des attributs ou clés qui sont enregistrés dans les fichiers.
 - Le débogage : le fonctionnement actuel pouvant être aléatoire, il peut être compliqué de reproduire certains bugs.
   Si un bug apparait selon l'ordre de définition des arguments, le nouveau comportement facilitera leur correction.
 - La priorité à donner aux arguments pourrait être spécifiée selon l'ordre de leur déclaration.
 - Les `namedtuple` pourraient être définis aisément avec une valeur par défaut.

Et probablement d'autres utilisations qui n'ont pas encore été envisagées.

# Proprités de classes



Toujours dans les petites modifications, un nouveau décorateur disponible de base devrait être introduit : `@classproperty`.
Si vous connaissez le modèle objet de Python, son nom devrait vous suffir pour deviner son but : permettre de définir des
propriétés au niveau de classes. Par exemple si vous souhaitez conserver au niveau de la classe le nombre d'instances
créées et des statistiques sur vos appels :

```python
class Spam:
    _n_instance_created = 0
    _n_egg_call = 0

    def __init__(self):
        self.__class__._n_instance_created += 1

    def egg(self):
        print("egg")
        self.__class__._n_egg_call += 1

    @classproperty
    def mean_egg_call(cls):
        return (cls._n_egg_call / cls._n_instance_created) if cls._n_instance_created > 0 else 0

spam = Spam()

spam.egg()
spam.egg()
spam.egg()

print(spam.mean_egg_call)   # => 3

spam2 = Spam()

# Les 3 expressions suivantes sont équivalentes
print(spam.mean_egg_call)    # => 1.5
print(spam2.mean_egg_call)   # => 1.5
print(Spam.mean_egg_call)    # => 1.5  , propriété au niveau de la classe !
```

Cet ajout a été proposé sur la *mailling-list python-ideas*. La mise en place d'une implémentation propre et complète de
ce comportement est compliquée sans définir une méta-classe. Une solution générique implique donc de le rajouter dans le
modèle objet de Python. Guido a rapidement donné son aprobation et un [ticket a été créé](http://bugs.python.org/issue24941)
en attendant qu'un des développeurs de CPython l'implémente dans le coeur de l'interpréteur. Cela pourrait donc être
l'une des premières nouvelles fonctionnalités confirmées pour la version 3.6.

# Sous-interpréteurs



[[a]]
| Cette section peut nécessiter des notions avancées de programmation système pour être comprise, en particulier la différence entre un *thread* et un *processus*, ainsi que leur gestion dans Python.

L'écriture de code concurrent en Python est toujours un sujet chaud. À ce jour, trois solutions existent dans la
bibliotèque standard :

 - *asyncio*, bien que limitée à un seul *thread*, permet d'écrire des coroutines s'exécutant en concurrence en tirant
 partie des temps d'attente introduits par les entrées/sorties.
 - *threading* permet de facilement exécuter du code sur plusieurs *threads*, mais leur exécution reste limitée à un seul
 coeur de processeur à cause du GIL.
 - *multiprocessing*, en *forkant* l'interpréteur, permet d'éxecuter plusieurs codes Python en parallèle sans limitation
 et exploitant pleinnement les ressources calculatoires des processeurs.

[[a]]
| Le [GIL](https://en.wikipedia.org/wiki/Global_Interpreter_Lock) est une construction implémentée dans de nompreux interpéteurs (CPython, Pypy, Ruby, etc.). Ce mécanisme bloque l'interpréteur pour qu'à chaque instant, un seul code puisse être exécuté. Ce sytème permet de s'assurer que du code éxécuté sur plusieurs *threads* ne va pas poser de problèmes de concurrence sur la mémoire, sans vraiment ralentir les codes n'utilisant qu'un seul *thread*. Malheureusement cela nous empèche d'exploiter les architectures multi-coeurs de nos processeurs.

La multiplicité des solutions ne résoud pas tout. En effet les limitation des *threads* en Python les rendent inutiles
quand le traitement exploite principalement le processeur. L'utilisation de *multiprocessing* est alors possible, mais
a un coût :

 - Le lancement d'un processus entraine un *fork* au niveau du système d'exploitation, ce qui prend plus de temps et de
 mémoire que le lancement d'un *thread* (presque gratuit en comparaison).
 - La communication entre les processus est aussi plus longue. Tandis que les *threads* permettent d'exploiter un
espace partagé en mémoire, rendant leurs communications directes, les processus nécessitent de mettre en place des mécanismes complexes, appelés
 [*IPC*](https://fr.wikipedia.org/wiki/Communication_inter-processus).

*[IPC]: Communication inter-processus

[Une proposition sur *python-ideas*](http://code.activestate.com/lists/python-ideas/34051/) a été formulée au début de
l'été pour offrir une nouvelle solution intermédiare entre les *threads* et les processus : des sous-interpréteurs
(*subinterpreters*), en espérant que cela puisse définitivement contenter les développeurs friands de codes concurrents.

À partir d'un nouveau module *subinterpreters*, reprenant l'interface exposée par les modules *threading* et *multiprocessing*,
CPython permettrait de lancer dans des *threads* différents interpréteurs, chacun chargé d'executer un code Python. Le fait
que ce soient des *threads* rendrait ce module beaucoup plus léger à utiliser que *multiprocessing*.
L'interpréteur se chargerait de lancer ces *threads* et partagerait le maximum de son implémentation entre eux.
Le plus intéressant est que ce mécanisme permettrait d'éluder le *GIL*. En effet, chaque sous-interpréteur disposerait
d'un espace de noms propre et indépendant. Chacun aurait donc un *GIL*, mais ceux-ci seraient indépendants. D'un point
de l'ensemble, la majorité des opérations transformeraient le *GIL* en *LIL* : *Local interpreter lock*. Chaque
sous-interpréteur pourrait ainsi fonctionner sur un coeur du processus séparé sans aucun problème, permettant ainsi au développeur de tirer
pleinement partie des processeurs modernes. Enfin, il faut savoir que CPython possède déjà en interne la notion de
sous-interpréteurs, le travail à réaliser consisterait donc à exposer ce code C pour le rendre disponible
en Python.

Évidement, cette proposition n'est pas une solution miracle. Tout n'est pas si simple et beaucoup d'élements doivent encore
être discutés. En particulier les moyens de communication entre les sous-interpréteurs (probablement via des queues, comme
pour *multiprocessing*) et tout un tas de petits détails d'implémentation. Mais la proposition a reçu un accueil très positif
et l'auteur est actuellement en train de préparer une PEP à ce sujet.

*[GIL]: *Global Interpreter Lock*

# Interpolation de chaines



Les interpolations directes de chaînes de caractères existent dans de nombreux langages : PHP, C#, Ruby, Swift, Perl, etc.
En voici un exemple en Perl :

```perl
my $a = 1;
my $b = 2;

print "Resultat = a + b = $a + $b = @{[$a+$b]}\n";
# Imprime "Resultat = a + b = 1 + 2 = 3"
```

Il n'existe pas d'équivalent direct en Python ; le plus proche pourrait ressembler à l'exemple suivant :

```python
a, b = 1, 2

print("Resultat = a + b = %d + %d = %d" % (a, b, a + b))
# ou
print("Resultat = a + b = {a} + {b} = {c}".format(a=a, b=b, c=a + b))
```

Nous voyons ainsi qu'il est nécessaire de passer explicitement les variables et, même s'il est possible d'utiliser
`locals()` ou `globals()` pour s'en passer, il n'est possible d'évaluer une expression, comme ici l'addition, 
qu'à l'extérieur de la chaîne de caractères puis d'injecter le résultat dans cette dernière.

La première proposition effectuée, formalisée par la [PEP 498](https://www.python.org/dev/peps/pep-0498/), propose de rajouter
cette possibilité dans Python grâce à un nouveau préfixe de chaîne, `f`, pour `format-string`, dont voici un exemple
d'utilisation, issu de la PEP :

```python
>>> import datetime
>>> name = 'Fred'
>>> age = 50
>>> anniversary = datetime.date(1991, 10, 12)
>>> f'My name is {name}, my age next year is {age+1}, my anniversary is {anniversary:%A, %B %d, %Y}.'
'My name is Fred, my age next year is 51, my anniversary is Saturday, October 12, 1991.'
>>> f'He said his name is {name!r}.'
"He said his name is 'Fred'."
```

Cette proposition a fait des émules. Plusieurs développeurs ont mit en évidence que la méthode d'interpolation peut
dépendre du contexte. Par exemple un module de base de données comme [`sqlite3`](https://docs.python.org/3.4/library/sqlite3.html)
propose un système de substitution personnalisé pour éviter les attaques par injection ; les moteurs de *templates*, eux, pour
produire du HTML, vont aussi faire des substitutions pour échapper certains caractères, comme remplacer `<` ou `>` par,
respectivement, `&lt;` ou `&gt;`. La [PEP 501](https://www.python.org/dev/peps/pep-0501/) propose donc aux développeurs de définir
leurs propres méthodes d'interpolation, adaptées au contexte.

Guido a déjà accepté la première proposition afin qu'elle soit implémentée dans Python 3.6. La deuxième est
plus générale mais plus complexe, et peut ainsi poser plusieurs problèmes. Aucune décision n'a encore été prise et les débats
à ce sujet ont encore lieu sur la *mailling list* concernant cette possibilité et la forme qu'elle pourrait prendre. Le *patch* pour la PEP 498 accepté est presque prêt et sera donc vraisemblablement la première nouvelle fonctionnalité de Python 3.6 ; l'implémentation de la PEP 501 dépendra de la suite des discussions.