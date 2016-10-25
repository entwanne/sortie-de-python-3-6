# Interpolation de chaînes littérales — PEP 498

Loin sont maintenant les `'My name is {name}'.format(name=name)` ou encore `'My name is {}'.format(name)`.
Avec Python 3.6, le formatage de chaînes de caractères gagne en clarté avec l'interpolation littérale.

En plus du préfixe `r` pour définir une chaîne brute, le préfixe `b` pour une chaîne d'octets, arrive maintenant le préfixe `f`, dédié au formatage.
Une chaîne préfixée de `f` sera interpolée à sa création, de manière à résoudre les noms utilisés au sein de la chaîne de formatage.

# Préservation de l'ordre des arguments nommés — PEP 468

# Protocole de gestion des chemins de fichiers — PEP 519

Cette PEP revient sur la `pathlib`, bibliothèque de gestion des chemins de fichiers, pour lui ajouter son propre protocole.

Spécialisée dans la manipulation de chemins de fichiers, cette bibliothèque comporte aussi de nombreux outils pour opérer sur ces fichiers (création de fichier, de dossier, suppression, etc.)

Une nouvelle méthode spéciale est maintenant disponible pour nos objets : la méthode `__fspath__`. Elle ne prend aucun paramètre et doit retourner une chaîne de caractères.
Tout objet implémentant cette méthode est considéré par Python comme un chemin, et peut alors être utilisé avec la `pathlib`.

# Préservation de l'ordre des attributs définis dans les classes — PEP 520

# Simplification de la personnalisation de classes — PEP 487

# Générateurs asynchrones — PEP 525
