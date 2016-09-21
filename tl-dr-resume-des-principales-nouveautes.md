
Les plus pressés peuvent profiter du court résumé [des principales nouveautés](https://docs.python.org/3.5/whatsnew/3.5.html) suivant. Chacune sera reprise dans la suite de l'article, en la détaillant et explicitant les concepts sous-jacents.

 - [PEP 448](https://www.python.org/dev/peps/pep-0448/) : les opérations d'*unpacking* sont généralisées et peuvent maintenant être combinées et utilisées plusieurs fois dans un appel de fonction.

    ```python
    >>> def spam(a, b, c, d, e, g, h, i, j):
    ...    return a + b + c + d + e + g + h + i + j
    ...
    >>> l1 = (1, *[2])
    >>> l1
    (1, 2)
    >>> d1 = {"j": 9, **{"i": 8}}
    >>> d1
    {"j": 9, "i": 8}
    >>> spam(*l1, 3, *(4, 5), g=6, **d1, **{'h':7})
    45    # 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9
    ```

 - [PEP 465](http://www.python.org/dev/peps/pep-0465) : l'opérateur binaire `@` est introduit pour gérer la multiplication matricielle et permet d'améliorer la lisibilité d'expressions mathématiques. Par exemple, l'équation $A \times B^T - C^{-1}$ correspondrait au code suivant avec `numpy`.

    ```python
    A @ B.T - inv(C)
    ```

 - [PEP 492](https://www.python.org/dev/peps/pep-0492) : les coroutines deviennent une construction spécifique du langage, avec l'introduction des mots-clés `async` et `await`. L'objectif étant de compléter le support de la programmation asynchrone dans Python, ils permettent d'écrire des coroutines utilisables avec `asyncio` de façon similaire à des fonctions Python synchrones classiques. Par exemple :

    ```python
    async def fetch_page(url, filename):
        # L'appel à aiohttp.request est asynchrone
        response = await aiohttp.request('GET', url)
        # Ici, on a récupéré le résultat de la requête
        assert response.status == 200
        async with aiofiles.open(filename, mode='wb') as fd:
            async for chunk in response.content.read_chunk(chunk_size):
                await fd.write(chunk)
    ```

 - [PEP 484](https://www.python.org/dev/peps/pep-0484/) : les annotations apposables sur les paramètres et la valeur de retour des fonctions et méthodes sont maintenant standardisées et ne devraient servir qu'à préciser le type de ces éléments. Les annotations ne sont toujours pas utilisées par l'interpréteur et cette PEP n'est constituée que de conventions.

    ```python
    def bonjour(nom: str) -> str:
        return 'Zestueusement ' + nom
    ```

