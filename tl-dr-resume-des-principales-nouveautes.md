Comme pour la version précédente, commençons par un résumé des [principales nouveautés](https://docs.python.org/3.6/whatsnew/3.6.html).
Les fonctionnalités listées ici seront bien sûr détaillées par la suite.

 - [PEP XXX](https://www.python.org/dev/peps/pep-XXX/) : les chaînes de caractères peuvent maintenant être préfixées du symbole `f` pour être interpolées en fonction des variables du scope courant.

    ```python
    >>> name = 'John'
    >>> f'Hello {name}!'
    'Hello John!'
    ```

 - [PEP XXX](https://www.python.org/dev/peps/pep-XXX/) : les arguments nommés reçus par une fonction sont maintenant assurés d'être ordonnés.

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

 - [PEP XXX](https://www.python.org/dev/peps/pep-XXX/) : le dictionnaire des attributs/méthodes définis dans le corps d'une classe est lui aussi ordonné.

 - [PEP XXX](https://www.python.org/dev/peps/pep-XXX/) : les descripteurs sont informés lorsqu'ils sont assignés à un attribut de la classe, *via* leur méthode `__set_name__`.

 - [PEP XXX](https://www.python.org/dev/peps/pep-XXX/) : une nouvelle méthode permet d'interférer sur la création de classes filles à la classe courante.
