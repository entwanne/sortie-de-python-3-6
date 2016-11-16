Comme pour la version précédente, commençons par un résumé des [principales nouveautés](https://docs.python.org/3.6/whatsnew/3.6.html).
Les fonctionnalités listées ici seront bien sûr détaillées par la suite.

 - [PEP 498](https://www.python.org/dev/peps/pep-0498/) : les chaînes de caractères peuvent maintenant être préfixées du symbole `f` pour être interpolées en fonction des variables du scope courant.

    ```python
    >>> name = 'John'
    >>> f'Hello {name}!'
    'Hello John!'
    ```

 - [PEP 468](https://www.python.org/dev/peps/pep-0468/) : les arguments nommés reçus par une fonction sont maintenant assurés d'être ordonnés.

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

 - [PEP 519](https://www.python.org/dev/peps/pep-0519/) : ajout d'un protocole de gestion des chemins de fichiers.

    ```python
    >>> import pathlib
    >>> path = pathlib.Path('/home')
    >>> path / 'john'
    PosixPath('/home/john')
    >>> with open(path / 'john' / '.python_history') as f:
    ...     history = f.read()
    ...
    ```

 - [PEP 520](https://www.python.org/dev/peps/pep-0520/) : le dictionnaire des attributs/méthodes définis dans le corps d'une classe est lui aussi ordonné.

    ```python
    >>> class A:
    ...     x = 0
    ...     y = 0
    ...     z = 0
    ...
    >>> A.__dict__
    mappingproxy({..., 'x': 0, 'y': 0, 'z': 0, ...})
    ```

 - [PEP 487](https://www.python.org/dev/peps/pep-0487/) : une nouvelle méthode (`__init_subclass__`) permet d'interférer sur la création de classes filles à la classe courante ; et les descripteurs sont informés lorsqu'ils sont assignés à un attribut de la classe, *via* leur méthode `__set_name__`.
