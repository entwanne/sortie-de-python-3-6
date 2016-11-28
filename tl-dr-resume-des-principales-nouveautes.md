Comme pour la version précédente, commençons par un résumé des [principales nouveautés](https://docs.python.org/3.6/whatsnew/3.6.html).
Les fonctionnalités listées ici seront bien sûr détaillées par la suite.

 -  [PEP 498](https://www.python.org/dev/peps/pep-0498/) : les chaînes de caractères peuvent maintenant être préfixées du symbole `f` pour être interpolées en fonction des variables du scope courant.

    Cette fonctionnalité reprend globalement la syntaxe supportée par la méthode `format` des chaînes de caractères.

    ```python
    >>> name = 'John'
    >>> f'Hello {name}!'
    'Hello John!'
    ```

 -  [PEP 468](https://www.python.org/dev/peps/pep-0468/) : les arguments nommés reçus par une fonction sont maintenant assurés d'être ordonnés.
    Le paramètre spécial `**kwargs` d'une fonction correspondra alors toujours à un dictionnaire ordonné des arguments nommés.

    ```python
    >>> def func(**kwargs):
    ...     for name, value in kwargs.items():
    ...         print(name, '->', value)
    ...
    >>> func(a=1, b=2, c=3)
    a -> 1
    b -> 2
    c -> 3
    >>> func(b=2, c=3, a=1)
    b -> 2
    c -> 3
    a -> 1
    ```

 -  [PEP 519](https://www.python.org/dev/peps/pep-0519/) : ajout d'un protocole pour les chemins de fichiers.
    Il devient maintenant plus facile de manipuler des chemins et d'interagir avec les fonctions système.

    ```python
    >>> import pathlib
    >>> path = pathlib.Path('/home')
    >>> path / 'john'
    PosixPath('/home/john')
    >>> with open(path / 'john' / '.python_history') as f:
    ...     history = f.read()
    ...
    ```

 -  [PEP 520](https://www.python.org/dev/peps/pep-0520/) : le dictionnaire de définition d'une classe devient lui aussi ordonné.
    L'ordre de définition des attributs et méthodes des classes est donc conservé.

    ```python
    >>> class A:
    ...     x = 0
    ...     y = 0
    ...     z = 0
    ...
    >>> A.__dict__
    mappingproxy({..., 'x': 0, 'y': 0, 'z': 0, ...})
    ```

 -  [PEP 487](https://www.python.org/dev/peps/pep-0487/) : une nouvelle méthode (`__init_subclass__`) permet d'interférer sur la création de classes filles à la classe courante ; et les descripteurs sont informés lorsqu'ils sont assignés à un attribut de la classe, *via* leur méthode `__set_name__`.

 -  [PEP 525](https://www.python.org/dev/peps/pep-0525/) : il devient possible d'écrire générateurs asynchrones.
    Ils fonctionneront comme des générateurs habituels, à utiliser depuis d'autres coroutines.

    ```python
    async def async_range(start, stop):
        for i in range(start, stop):
            yield i
    ```

- [PEP 530](https://www.python.org/dev/peps/pep-0530/) : le mot-clé `async` peut maintenant être utilisé dans les listes en intension (et autres compréhensions) au sein d'une coroutine.

    ```python
    async def coroutine():
        return [i async for i in async_range(0, 10)]
    ```
