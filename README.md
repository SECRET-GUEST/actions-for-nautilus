# Actions Pour Nautilus
Une extension pour le gestionnaire de fichiers Gnome **Files** (également connu sous le nom de Nautilus) qui vous permet d'ajouter des actions arbitraires au menu contextuel de sélection de Files.

Cette extension est un "remplaçant" pour la fonctionnalité désormais obsolète du gestionnaire de fichiers Nautilus du projet `filemanager/nautilus-actions`.

L'extension prend en charge de nombreuses fonctionnalités les plus couramment utilisées du projet d'extension d'origine, y compris :

* structurer des éléments de menu contextuel pour les sélections du gestionnaire de fichiers Nautilus, y compris les sous-menus imbriqués
* filtrer les éléments affichés en fonction de :
  * nombre de fichiers dans la sélection,
  * permissions d'accès de l'utilisateur pour les fichiers sélectionnés,
  * mimetypes des fichiers sélectionnés (conditions correspondantes et non correspondantes supportées, ainsi que les motifs de mimetypes),
  * types de fichiers basiques des fichiers sélectionnés - par exemple 'fichier', 'répertoire', 'lien symbolique'... - (conditions correspondantes et non correspondantes supportées),
  * correspondance des motifs de chemin complet, exprimés comme motifs glob ou expressions régulières, là encore avec support pour conditions correspondantes et non correspondantes.
* exécution d'une commande/script arbitraire lorsqu'un élément de menu est activé, avec les mêmes sémantiques "PLURAL" et "SINGULAR" que le projet `filemanager/nautilus-actions`
* support pour tous les placeholders de ligne de commande implémentés par le projet `filemanager/nautilus-actions`, avec les mêmes sémantiques.

Elle est également _beaucoup_ plus efficace pour exécuter des commandes dans un shell que l'extension d'origine, permettant la construction de pipelines et de boucles, ainsi que l'utilisation d'expressions shell plus complexes, sans avoir besoin d'écrire des scripts enveloppants.

[Une application de configuration](#configuration-ui) du nom de "Actions Pour Nautilus Configurator" est installée dans votre collection d'applications de bureau. Lorsque vous utilisez le configurateur pour la première fois, si aucun fichier de configuration existant n'est trouvé, la [configuration d'exemple](./configurator/sample-config.json) fournie sera installée.

Le projet dispose d'un [wiki](https://github.com/bassmanitram/actions-for-nautilus/wiki) qui est utilisé pour partager des astuces et des exemples de configuration utiles.


## 📋 Sommaire

1. [Installation](#installation)
2. [Référence de Configuration](#référence-de-Configuration)
3. [Place holders](#place-holders)
6. [Comportement d'exécution](#comportement-dexecution)
7. [Diagnostics](#diagnostics)
8. [Remerciements](#remerciements)


# Installation
## Systèmes basés sur Debian

Des paquets Debian des versions les plus récentes sont fournis dans le dossier [dist](./dist).

Téléchargez simplement le paquet, installez-le avec votre installateur de paquets, puis lancez l'application **Actions Pour Nautilus Configurator** depuis la liste de vos applications pour commencer à construire une configuration basée sur l'[exemple](#sample-configuration) fourni.

Pour activer l'extension après l'installation, vous devrez redémarrer Nautilus/Files :

* `Alt F2`
* `nautilus -q`

devrait suffire.

### Paquets Additionnels Suggérés
Le paquet Debian spécifie les dépendances suivantes comme **Suggère**, qui augmenteront grandement l'utilité de l'extension ainsi que permettront à la configuration d'exemple fournie de fonctionner dès le premier lancement :

* `xclip`  - un outil en ligne de commande pour gérer les presse-papiers X
* `zenity` - une boîte à outils d'interface utilisateur Gnome pour les scripts shell

Il est fortement recommandé d'installer ces paquets supplémentaires.

## Installation Manuelle
### Installer les Dépendances

Premièrement, bien sûr, l'extension dépend de GNOME et GNOME Files (alias Nautilus) étant installés.

Ensuite, elle dépend de `python 3+`, `nautilus-python`, et certains outils de gestion de processus (qui sont probablement déjà installés mais au cas où :)).

* Fedora `sudo dnf install nautilus-python python3-gobject procps-ng js-jquery`
* Ubuntu `sudo apt install python3-nautilus python3-gi procps libjs-jquery`
* Arch `sudo pacman -S python-nautilus python-gobject procps-ng jquery`

### Télécharger & Installer l'Extension

Pour installer l'extension manuellement, vous devrez suivre ces étapes :

1. `git clone https://github.com/bassmanitram/actions-for-nautilus.git`
2. `cd actions-for-nautilus`
3. `make install`

 pour installer uniquement pour votre utilisation, ou `sudo make install_global` pour installer pour tous les utilisateurs.
5. Vous _pourriez_ devoir redémarrer la coquille Gnome pour voir l'application de configuration dans votre liste d'applications de bureau

Si vous n'avez pas les commandes `git` ou `make` dans votre système, installez-les simplement de la même manière que vous avez installé les [autres dépendances](#install-dependencies).

Lors de la _première_ installation, vous ne verrez rien de différent dans les menus contextuels de Nautilus, car vous avez besoin d'avoir une configuration fonctionnelle pour que quelque chose change. La configuration d'exemple sera installée pour l'utilisateur simplement en démarrant l'[interface utilisateur de configuration](#configuration-ui).

### Désinstallation

1. `cd path/to/actions-for-nautilus`   
2. `make uninstall` si vous avez installé uniquement pour votre utilisation, ou `sudo make uninstall_global` si vous avez installé pour tous les utilisateurs.
3. Vous _pourriez_ devoir redémarrer la coquille Gnome pour enlever l'application de configuration de votre liste d'applications de bureau


## Configuration d'exemple
Le [fichier de configuration d'exemple](./configurator/sample-config.json) fourni est copié à 

```
${HOME}/.local/share/actions-for-nautilus/config.json
```

lorsque vous démarrez l'interface utilisateur de configuration pour la première fois, s'il n'y a pas de configuration existante.

La configuration contient des exemples de construction de commandes et de menus, y compris :

* des sous-menus contextuels,
* conditions de mimetype, de type de fichier, et de nombre de sélection,
* l'utilisation de pipelines de commandes,
* exploitation des substitutions de commandes et d'arguments `$(...)`/backtick.
* ...

Les commandes configurées dépendent de quelques dépendances supplémentaires qui doivent être installées si vous voulez voir la configuration d'exemple fonctionner correctement :

* `gedit` - l'éditeur standard Gnome - vous l'avez probablement déjà
* `gnome-terminal` - l'émulateur de terminal standard Gnome (pour l'instant) - vous l'avez probablement aussi.
* `xclip`  - un outil en ligne de commande pour gérer les presse-papiers X
* `zenity` - une boîte à outils d'interface utilisateur Gnome pour les scripts shell

Ces outils peuvent également être installés en utilisant le gestionnaire de paquets de votre plateforme comme indiqué ci-dessus.

Il est également possible que la sémantique des structures de commandes plus complexes repose sur des fonctionnalités de shell qui, si vous n'utilisez pas BASH comme shell système, ne fonctionneront pas pour vous.


### Le Profil "Pas de Fermeture" de Gnome Terminal
Lors de l'exécution de la commande `gnome-terminal`, la configuration d'exemple fait référence à un profil `gnome-terminal` nommé "Pas de Fermeture".

Il ne s'agit pas d'un profil standard, mais c'est utile de le définir car le terminal ne se ferme pas lorsque la commande qu'il exécute se termine, vous permettant de voir la sortie de la commande et/ou de relancer la commande.

Vous pouvez créer ce profil comme suit :
  
* Ouvrez l'application `gnome-terminal`
* Trouvez la boîte de dialogue `Préférences` (soit un élément de menu ou cliquez sur le bouton `...`, puis sur `Préférences`)
* Cliquez sur le **+** à côté du mot `Profils`
* Donnez à ce nouveau profil le nom `Pas de Fermeture`
* Cliquez sur l'onglet `Commande`
* Assurez-vous que l'option `Lorsque la commande se termine :` est réglée sur `Maintenir le terminal ouvert`
* Configurez tout autre paramètre nécessaire concernant le comportement et l'apparence du profil

# Interface de Configuration
Lorsque vous installez cette extension, une application de configuration est installée dans votre collection d'applications de bureau locale.

Pour démarrer l'application :
* Ouvrez le navigateur de votre collection d'applications (menu, panneau, ...)
* Trouvez **Actions Pour Nautilus Configurator**
* Cliquez dessus

L'application s'ouvrira dans votre navigateur Web par défaut. Elle présentera la configuration actuelle (en créant une à partir de [l'exemple fourni](./configurator/sample-config.json) si aucune configuration n'existe encore pour l'utilisateur).

L'interface _devrait_ être assez explicite - vous pouvez ajouter, supprimer, déplacer et modifier les Menus et les Commandes à votre guise.

Il y a aussi un éditeur de source JSON intégré avec vérification de la syntaxe si vous souhaitez effectuer des actions non supportées par l'interface principale (comme copier/coller des actions).

Fermez simplement la page web pour quitter le configurateur.

*NOTE* l'application web de configuration NE communique JAMAIS en dehors de votre propre système, à moins que vous ne cliquiez sur un lien externe référencé dans les informations d'aide.

## Aide du Configurateur
L'interface utilisateur inclut une aide intégrée accessible de deux façons :

* Cliquez sur le bouton `Afficher l'Aide` pour ouvrir le volet d'aide à droite de l'interface principale du configurateur, positionné au début des informations d'aide.
* Cliquez sur l'une des icônes &#9432; pour ouvrir le volet d'aide à droite de l'interface principale du configurateur, positionné sur les informations relatives à l'élément de l'interface utilisateur auquel l'icône &#9432; est attachée.

Quand le configurateur et les informations d'aide sont affichés, la taille des volets peut être ajustée en faisant glisser la ligne qui les sépare.

Pour fermer les informations d'aide, cliquez simplement sur le bouton `Masquer l'Aide` (qui est le bouton `Afficher l'Aide` avec le label modifié lorsque les informations d'aide sont affichées).

## Sauvegarder vos changements
Pour sauvegarder les changements de configuration, cliquez sur le bouton **Sauvegarder Config**. Vos changements devraient être visibles dans Nautilus après environ 30 secondes (le délai d'expiration pour le surveillant interne de changement de fichier de config).

Le fichier de configuration existant est sauvegardé avant d'être écrasé par une configuration sauvegardée. Vous pouvez rétablir une ancienne configuration en ouvrant Nautilus/Files, en naviguant vers le dossier...

```
${HOME}/.local/share/nautilus-python/extensions/actions-for-nautilus
```

... et en remplaçant votre fichier `config.json` actuel par l'une des copies sauvegardées. Encore une fois, les changements prendront effet après un maximum d'environ 30 secondes.

# Référence de Configuration

[Retour au Sommaire](#-sommaire)

La configuration est spécifiée dans un fichier texte JSON nommé `config.json` situé à

```
${HOME}/.local/share/actions-for-nautilus
```

L'extension est livrée avec un strict valide [Schéma JSON](./configurator/actions-for-nautilus.schema.json) qui décrit exactement comment le fichier de configuration doit être construit.

(Notez qu'il y a aussi un _deuxième_ [Schéma JSON](./configurator/actions-for-nautilus.ui.schema.json) livré. Celui-ci est pour usage interne par le configurateur et ne devrait pas être considéré comme une description canonique du fichier de configuration de l'extension).

## Structure de niveau supérieur
La structure de niveau supérieur dans le fichier de configuration doit être un objet JSON qui est censé contenir une propriété nommée `actions` dont la valeur est, elle-même, un tableau d'objets, et une propriété de chaîne nommée `sort` :


```
{
  "actions": [
    {
      ...
    },
    "sort": "manual or auto"
  ]
}
```

La propriété `sort` est optionnelle et indique la méthode à utiliser pour trier les actions présentées par le menu de niveau supérieur. Les valeurs autorisées sont :
* `manual` - L'extension laisse les éléments dans l'ordre dans lequel ils apparaissent dans la configuration
* `auto` - L'extension trie les éléments par ordre alphanumérique

La valeur par défaut est - `manual`

Le tableau `actions` contient la configuration de chaque action à présenter dans le menu de niveau supérieur.

Chaque élément du tableau est alors un objet (et *action*) qui, principalement, doit avoir une propriété nommée `type` dont la valeur est soit `command` soit `menu`, et une propriété nommée `label` dont la valeur est le texte que vous souhaitez voir dans le menu contextuel de Nautilus.

```
    {
      "type": "command",
      "label": "My Command",
      ...
    },
    {
      "type": "menu",
      "label": "My Sub Menu",
      ...
    },
    ...

```
Les sections suivantes décrivent en détail ces objets d'action.

## Actions de menu
Les actions avec une propriété `type` de `menu` définissent des actions de "sous-menu" qui, lorsqu'on clique dessus, exposent un menu imbriqué de davantage d'actions, celles-ci étant des actions de commande ou d'autres menus imbriqués.

```
    ...
    {
      "type": "menu",
      "label": "My Sub Menu",
      "actions": [
        ...
      ],
      "sort": "manual or auto"
    },
    ...
```

Les actions de menu sont censées contenir deux propriétés supplémentaires :

* `actions` - OBLIGATOIRE - un tableau d'éléments dont chacun suit le même modèle que les éléments contenus par la propriété `actions` à la racine de la configuration

* `sort` - FACULTATIF - L'approche à utiliser pour trier les actions présentées par le menu
   * `manual` - L'extension laisse les éléments dans l'ordre dans lequel ils apparaissent dans la configuration
   * `auto` - L'extension trie les éléments par ordre alphanumérique
   
  *Par défaut* - `manual`

Lorsque le menu contextuel de Nautilus/Files est activé pour une sélection, l'extension évalue toutes les commandes configurées au sein d'un menu pour déterminer si les commandes sont pertinentes pour la sélection actuelle. Si aucune commande n'est jugée pertinente, alors le menu n'apparaît pas dans le menu contextuel de Nautilus/Files.

## Actions de commande
Les actions avec une propriété `type` de `command` définissent des actions qui, lorsqu'on clique dessus, exécutent une commande.

```
    ...
    {
      "type": "command",
      "label": "My Command",
      command_line: "my-script.sh %F %c",
      cwd: "%d",
      use_shell: true,
      min_items: 1,
      max_items: 1,
      "mimetypes": [
        ...
      ],
      "filetypes": [
        ...
      ],
      "path_patterns": [
        ...
      ]
    },
    ...
```
Celles-ci sont supposées avoir les propriétés supplémentaires suivantes :

* `command_line` - OBLIGATOIRE - la commande système à exécuter lorsque l'élément de menu est cliqué, exprimé comme une chaîne de caractères.

  La commande peut contenir des expressions réservées qui sont développées pour détenir les détails des fichiers sélectionnés qui sont passés comme arguments à la commande.

  Le projet `filemanager/nautilus-actions` prend en charge l'ensemble complet de réserves, avec les mêmes sémantiques, elles sont documentées plus loin.

  Notez qu'en utilisant l'option `use_shell` (ci-dessous), la ligne de commande peut être presque tout ce que vous pouvez entrer dans un prompt shell, y compris les fonctionnalités suivantes :
  
  * Pipelines
  * Génération/expansion de commandes et d'arguments `$(...)` ou "backtick"
  * Résolution de variables d'environnement
  * Boucles
  * ...
  
  Consultez la [configuration d'exemple](./configurator/sample-config.json) pour quelques exemples.
  
* `cwd` - FACULTATIF - le répertoire de travail dans lequel la commande doit "s'exécuter", exprimé comme une chaîne de caractères.

  Cela peut aussi contenir des expressions réservées, bien qu'elles doivent résoudre en un seul nom de répertoire valide.
  
  *Par défaut* - non défini
  
* `use_shell` - FACULTATIF - une valeur booléenne (`true` ou `false`) indiquant si la commande doit être exécutée par le shell système par défaut. Si la commande est un script shell, ou dépend de toute sémantique d'expansion shell, vous devez régler la valeur de cette propriété sur `true`.

  *Par défaut* - `false`
  
* `filetypes` - FACULTATIF - les types de fichiers généraux des fichiers sélectionnés pour lesquels cette action doit être affichée (ou pour lesquels l'action ne doit pas être affichée).

  La valeur doit être une liste JSON de chaînes dont chacune doit avoir une des valeurs suivantes :
  
  * `unknown` - pour les fichiers d'un type inconnu
  * `directory` - pour les répertoires
  * `file` - pour les fichiers standard
  * `symbolic-link` - pour les liens symboliques
  * `special` - pour les fichiers spéciaux (pipes, périphériques, ...)
  * `standard` - raccourci pour les répertoires, les fichiers standard et les liens symboliques

  De nouveau, ceux-ci peuvent être préfixés par un caractère `!` pour indiquer que les fichiers sélectionnés ne doivent _pas_ être de ce type.

  Seule la première apparition d'un type de fichier spécifique (indépendamment de tout préfixe `!` "non") est prise en compte.

  *Par défaut* - tous les types de fichiers sont acceptés

* `min_items` - FACULTATIF - le nombre minimum d'éléments dans la sélection pour lesquels cette action sera affichée.

  Par exemple, si la commande doit, disons, comparer un certain nombre de fichiers, cela n'a pas de sens que l'action soit affichée lorsque moins de deux fichiers sont dans la sélection. Dans ce cas, vous définiriez la valeur de cette propriété à `2`, ce qui empêcherait l'action d'apparaître dans le menu contextuel lorsqu'un seul fichier est dans la sélection.

  Si spécifié, la valeur doit être supérieure à zéro.

  Si la valeur de `max_items` est supérieure à zéro, la valeur de cette propriété doit être inférieure ou égale à la valeur de `max_items`.

  *Par défaut* - 1

* `max_items` - FACULTATIF - le nombre maximum d'éléments dans la sélection pour lesquels cette action sera affichée.

  Par exemple, si la commande doit, disons, démarrer un serveur HTTP dans un répertoire sélectionné, cela n'a pas de sens que l'action soit affichée lorsque plus d'un répertoire est dans la sélection. Donc, dans ce cas, vous définiriez la valeur de cette propriété à `1`, ce qui empêcherait l'action d'apparaître dans le menu contextuel lorsque plus d'un répertoire est dans la sélection.

  Une valeur de zéro signifie `illimité`.

  Si la valeur est supérieure à zéro, la valeur de la propriété `min_items` doit être inférieure ou égale à cette valeur.

  *Par défaut* - illimité

* `mimetypes` - FACULTATIF - les types MIME des fichiers sélectionnés pour lesquels cette action doit être affichée (ou pour lesquels l'action ne doit pas être affichée).

  La valeur doit être une liste JSON de chaînes dans le format suivant :

  * `*/*` ou `*

` - signifiant que l'action peut être affichée pour tous les types MIME
  * `type/sous-type` - pour afficher l'action pour les fichiers d'un type MIME spécifique
  * `type/*` - pour afficher l'action pour les fichiers dont les types MIME sont n'importe quel sous-type d'un type spécifique
  * `!type/sous-type` - pour _ne pas_ afficher l'action pour les fichiers d'un type MIME spécifique
  * `!type/*` - pour _ne pas_ afficher l'action pour les fichiers dont les types MIME sont n'importe quel sous-type d'un type spécifique

  Tous les fichiers dans la sélection doivent correspondre aux règles de type MIME d'une action pour que cette action soit affichée. Mélanger des règles "non" avec des règles "non non", peut être déroutant.

  Seule la première apparition d'une règle spécifique (indépendamment de tout préfixe `!` "non") est prise en compte.

  *Par défaut* - tous les types MIME sont acceptés

* `path_patterns` - FACULTATIF - une liste de motifs globaux ou d'expressions régulières contre lesquels les chemins complets des fichiers sélectionnés doivent être comparés.

  La valeur doit être une liste JSON de chaînes, chacune dans l'un des formats suivants :

  * une expression "glob" - une syntaxe d'expression de motif de chaîne simple mais limitée qui est utilisée par de nombreuses commandes shell UNIX ainsi que par le shell lui-même, composée des réserves suivantes :
    
    * `*` indiquant zéro ou plusieurs caractères
    * `?` indiquant un seul caractère
    * `[abc]` indiquant l'un des caractères entre les crochets
    * `[!abc]` indiquant aucun des caractères entre les crochets

    Très souvent, cette syntaxe est tout ce dont vous avez besoin pour exprimer le motif que vous souhaitez comparer.

    Notez que les globs correspondent intrinsèquement à l'ensemble du chemin.

  * `re:` suivi d'une expression régulière (SANS délimiteurs `/`) - des besoins plus complexes peuvent être exprimés en expressions régulières.

    Notez que les expressions régulières _ne correspondent pas_ intrinsèquement à l'ensemble du chemin.
    
    Cela signifie que si une partie d'un chemin de fichier sélectionné correspond à l'expression régulière, le chemin sera accepté.

    Si vous souhaitez correspondre à l'ensemble du chemin, commencez votre expression régulière par `^` et terminez-la par `$`.
   
  Chaque format de motif peut être préfixé par `!` pour nier le motif.

  Tous les fichiers dans la sélection doivent correspondre aux règles de motif de chemin d'une action pour que cette action soit affichée. Mélanger des règles "non" avec des règles "non non", peut être déroutant.

  Seule la première apparition d'une règle spécifique (indépendamment de tout préfixe `!` "non") est prise en compte.

  La syntaxe glob acceptée est entièrement documentée [ici](https://docs.python.org/3/library/fnmatch.html).
  La syntaxe d'expression régulière acceptée est entièrement documentée [ici](https://docs.python.org/3/library/re.html#regular-expression-syntax).

  *Par défaut* - tous les chemins de fichiers sont acceptés

* `permissions` - FACULTATIF - un indicateur des permissions d'accès minimales que l'utilisateur doit avoir pour les fichiers sélectionnés afin que l'action associée soit présentée dans le menu contextuel de Nautilus.

  Les valeurs valides sont :

  * `read` - l'utilisateur doit au moins avoir des permissions de lecture pour les fichiers sélectionnés
  * `read-write` - l'utilisateur doit au moins avoir des permissions de lecture et d'écriture pour les fichiers sélectionnés
  * `read-execute` - Pour les fichiers, l'utilisateur doit au moins avoir des permissions de lecture et d'exécution pour les fichiers sélectionnés. Pour les dossiers, l'utilisateur doit au moins avoir des permissions de lecture et de navigation pour les dossiers sélectionnés
  * `read-write-execute` - l'utilisateur doit avoir des permissions complètes de lecture, d'écriture et d'exécution/navigation pour les fichiers sélectionnés

  Toute autre valeur désactivera la vérification des permissions.

  *Par défaut* - les permissions d'accès de l'utilisateur ne sont pas vérifiées.

Avec les list

es de filtres `mimetypes`, `filetypes` et `path_patterns`, tous les fichiers sélectionnés doivent correspondre à au moins une règle non niée (s'il y a des règles non niées), tout en ne correspondant à aucune des règles niées, pour que l'action associée apparaisse dans le menu contextuel.

# Place holders

[Retour au Sommaire](#-sommaire)

Tous les détenteurs de place de ligne de commande et `cwd` mis en œuvre par le projet `filemanager/nautilus-actions` sont implémentés par cette extension, avec les mêmes sémantiques :

| Marqueur    | Description                                                                                                      | Répétition |
|-------------|------------------------------------------------------------------------------------------------------------------|------------|
| `%b`        | le nom de base du premier élément sélectionné (par exemple, `mon-fichier.txt`)                                   | SINGULIER |
| `%B`        | liste séparée par des espaces des valeurs `%b` de tous les éléments sélectionnés                                | PLURIEL    |
| `%c`        | le nombre d'éléments dans la sélection                                                                          | TOUT       |
| `%d`        | le chemin complet vers le répertoire contenant le premier élément sélectionné (par exemple, `/home/moi/mon-premier-repertoire/mon-deuxieme-repertoire`) | SINGULIER |
| `%D`        | liste séparée par des espaces des valeurs `%d` de tous les éléments sélectionnés                                | PLURIEL    |
| `%f`        | le chemin complet du premier élément sélectionné (par exemple, `/home/moi/mon-premier-repertoire/mon-deuxieme-repertoire/mon-fichier.txt`) | SINGULIER |
| `%F`        | liste séparée par des espaces des valeurs `%f` de tous les éléments sélectionnés                                | PLURIEL    |
| `%h`        | le nom d'hôte à partir de l'URI du premier élément sélectionné                                                | TOUT       |
| `%m`        | le type MIME du premier élément sélectionné (par exemple, `text/plain`)                                         | SINGULIER |
| `%M`        | liste séparée par des espaces des valeurs `%m` de tous les éléments sélectionnés                                | PLURIEL    |
| `%n`        | le nom d'utilisateur à partir de l'URI du premier élément sélectionné                                          | TOUT       |
| `%o`        | opérateur non-opérationnel qui force une forme d'exécution SINGULIERE - voir ci-dessous pour plus de détails   | SINGULIER |
| `%O`        | opérateur non-opérationnel qui force une forme d'exécution PLURIELLE - voir ci-dessous pour plus de détails    | PLURIEL    |
| `%p`        | le port à partir de l'URI du premier élément sélectionné                                                       | TOUT       |
| `%s`        | le schéma URI à partir de l'URI du premier élément sélectionné (par exemple, `file`)                           | TOUT       |
| `%u`        | l'URI du premier élément sélectionné (par exemple, `file:///home/moi/mon-premier-repertoire/mon-deuxieme-repertoire/mon-fichier.txt`) | SINGULIER |
| `%U`        | liste séparée par des espaces des valeurs `%u` de tous les éléments sélectionnés                                | PLURIEL    |
| `%w`        | le nom de base du premier élément sélectionné sans son extension (par exemple, `mon-fichier`)                  | SINGULIER |
| `%W`        | liste séparée par des espaces des valeurs `%w` de tous les éléments sélectionnés                                | PLURIEL    |
| `%x`        | l'extension du premier élément sélectionné sans son extension (par exemple, `txt`)                              | SINGULIER |
| `%X`        | liste séparée par des espaces des valeurs `%x` de tous les éléments sélectionnés                                | PLURIEL    |
| `%%`        | le caractère `%`                                                                                                | TOUT       |
Tout espace intégré trouvé dans les valeurs individuelles est 'échappé' pour assurer que le shell ou le système reconnaît chaque valeur comme un argument indépendant et complet à la commande.

La signification de la valeur `Répétition` est expliquée dans la section suivante.

# Comportement d'exécution 

[Retour au Sommaire](#-sommaire)

Le projet `filemanager/nautilus-actions` a implémenté une fonctionnalité par laquelle une commande configurée pourrait être exécutée une seule fois, quel que soit le nombre d'éléments dans la sélection, ou une fois pour chaque élément dans la sélection.

Cette extension implémente la même fonctionnalité avec les mêmes sémantiques.

La décision quant au mode souhaité est basée sur le premier marqueur trouvé dans la valeur de propriété `command_line` pour l'action activée:

* Si le marqueur a une propriété `Répétition` de `SINGULAIRE`, la commande est exécutée une fois pour chaque élément dans la sélection.
* Si le marqueur a une propriété `Répétition` de `PLURIEL`, la commande est exécutée une seule fois.
* Si le marqueur a une propriété `Répétition` de `TOUT`, alors le _prochain_ marqueur est examiné.
* Si aucun marqueur avec une valeur de répétition `SINGULAIRE` ou `PLURIEL` n'est trouvé dans la commande, alors la commande est exécutée une seule fois.

De plus, si la commande doit être exécutée une fois pour chaque élément dans la sélection, alors n'importe quel marqueur avec une valeur de `Répétition` de `SINGULAIRE` est résolu à la valeur correspondante pour l'élément sélectionné pour lequel la commande est exécutée.

Les marqueurs avec des valeurs `Répétition` qui ne sont pas `SINGULAIRE` sont résolus à leurs valeurs complètes pour chaque exécution de la commande.

## Un exemple

Cet exemple est directement tiré de la documentation du projet `filemanager/nautilus-actions`:

> Disons que le dossier actuel est `/data`, et la sélection actuelle contient les trois fichiers `pierre`, `paul` et `jacques`.
> 
> Si nous avons demandé `echo %b`, alors les commandes suivantes seront successivement exécutées :
> 
> ```
> echo pierre
> echo paul
> echo jacques
> ```
> 
> Ceci parce que `%b` marque un paramètre SINGULIER. La commande est alors exécutée une fois pour chacun des éléments sélectionnés.
> 
> À l'inverse, si nous avons demandé `echo %B`, alors la commande suivante sera exécutée :
> 
> ```
> echo pierre paul jacques
> ```
> 
> Ceci parce que `%B` marque un paramètre PLURIEL. La commande est alors exécutée une seule fois, avec la liste des éléments sélectionnés comme arguments.
> 
> Si nous avons demandé `echo %b %B`, alors les commandes suivantes seront successivement exécutées :
> 
> ```
> echo pierre pierre paul jacques
> echo paul pierre paul jacques
> echo jacques pierre paul jacques
> ```
> 
> Ceci parce que le premier paramètre pertinent est `%b`, et ainsi la commande est exécutée une fois pour chaque élément sélectionné, remplaçant à chaque occurrence le paramètre `%b` avec l'élément correspondant. Le second paramètre est calculé et ajouté comme arguments à la commande exécutée.
> 
> Et si nous avons demandé `echo %B %b`, alors la commande suivante sera exécutée :
> 
> ```
> echo pierre paul jacques pierre
> ```
> 
> Ceci parce que le premier paramètre pertinent ici est `%B`. La commande est alors exécutée une seule fois, remplaçant `%B` avec la liste séparée par des espaces des noms de base. Comme la commande n'est exécutée qu'une seule fois, le `%b` est substitué une seule fois avec le (premier) nom de base.

# Diagnostics

[Retour au Sommaire](#-sommaire)

Les messages d'erreur sont envoyés à la sortie standard (`stdout`) ou à la sortie d'erreur (`stderr`) de Nautilus - y compris les erreurs trouvées dans le fichier de configuration (telles qu'un format JSON invalide).

De plus, la propriété `debug` peut être définie dans l'objet de niveau supérieur, avec une valeur de `true` ou `false` (par défaut). Lorsqu'il est défini sur `true`, d'autres informations de débogage sont imprimées sur le `stdout` de Nautilus.

Pour _voir_ cette sortie, vous devrez démarrer Nautilus d'une manière spéciale depuis un émulateur de terminal (par exemple, `gnome-terminal`):

```
# Arrêter Nautilus
nautilus -q  
# Redémarrer avec `stdout` et `stderr` affichés sur le terminal
nautilus --no-desktop
```

Notez que, pour arrêter ce mode d'exécution spécial, vous devrez soit fermer l'émulateur de terminal, soit, depuis un autre émulateur, exécuter la commande `nautilus -q`.

# Remerciements

[Retour au Sommaire](#-sommaire)

La principale reconnaissance est, bien sûr, à l'extension Nautilus Actions originale, plus tard renommée [Actions du gestionnaire de fichiers](https://gitlab.gnome.org/Archive/filemanager-actions) pour refléter son applicabilité plus large (Nemo, par exemple).

Malheureusement, cette extension n'est plus maintenue et n'est plus fonctionnelle depuis Nautilus 42.2 (lui-même maintenant renommé Gnome Files, bien que les objets de programmation sous-jacents soient toujours dans l'espace de noms Nautilus).

J'ai été tenté de reprendre la maintenance de ce projet, mais j'ai été rebuté par son implémentation en C complexe (je suis un programmeur en C parfaitement compétent, notez bien !).

J'étais convaincu qu'une implémentation beaucoup moins complexe de la plupart des fonctionnalités principales était possible en utilisant Python et le liant à Nautilus trouvé dans le framework `nautilus-python`, et en utilisant un format de configuration beaucoup plus sémantiquement pertinent tel que JSON et en adaptant un éditeur JSON existant plutôt que de construire une UI de configuration à partir de zéro.

Je pense avoir prouvé mon point de vue :)

Un autre grand remerciement est à [Christoforos Aslanov](https://github.com/chr314) dont l'extension [Nautilus Copy Path](https://github.com

/chr314/nautilus-copy-path) a fourni l'inspiration et le modèle pour le POC original de cette extension, et dont la structure de projet, la procédure d'installation et la documentation que j'ai initialement déchirée sans pitié :)... et je suis même assez irrespectueux pour avoir fourni une alternative à son extension dans ma propre configuration d'échantillon !

Merci et excuses, Christoforos.

L'éditeur basé sur le schéma JSON [JSON-Editor](https://github.com/json-editor/json-editor) est une trouvaille incroyable ! Le configurateur est, en effet, une instance de cet éditeur avec quelques ajustements pour le rendre un peu plus naturel pour ce cas d'utilisation !

L'éditeur de source JSON intégré est l'éditeur de source [ACE](https://ace.c9.io/) - un autre projet incroyable qui était si facile à intégrer qu'on se demande pourquoi JSON-Editor ne l'utilise pas pour sa propre fonctionnalité d'édition de source JSON - je sens un PR à venir :).

Alors, un GRAND cri à ces deux projets !
