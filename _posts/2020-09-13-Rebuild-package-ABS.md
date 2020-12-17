---
title: Recompiler Vim avec Arch Build System
excerpt: Utilisation d'Arch Build System pour recompiler un paquet existant pour le modifier et l'adapter à ses besoins.
tags: [ABS, makepkg, PKGBUILD]
categories: Articles
comments: true
---


# Introduction

Lors de l'installation du plugin d'autocompletion [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe), j'ai rencontré l'erreur suivante au démarrage de `Vim` :  
```
$ vim file.py
YouCompleteMe unavailable: unable to load Python.
Press ENTER or type command to continue
```  

> Au cours de ce billet, nous allons aborder plusieurs points :
> * Comment vérifier les modules Python supportés par `Vim`
> * Comment télécharger les fichiers sources d'un paquet existant
> * Comment modifier un fichier PKGBUILD
> * Comment build un paquet à l'aide de `makepkg`
> * Comment gérer les conflits lors des mises à jours du système

## Vérification des modules Python supportés par Vim

Au final, il s'avère que ma version de `Vim` était compilée pour supporter Python 2 et Python 3. Hors, YouCompleteMe ne supporte plus Python 2, d'où l'apparition de l'erreur.  
<br>
Il est possible de vérifier cela grâce à la commande `vim --version`, qui permet de voir les fonctionnalités qui sont inclues ou non dans notre version de `Vim` : 

```
$ vim --version 
VIM - Vi IMproved 8.2 (2019 Dec 12, compiled Sep 07 2020 21:40:38)
Included patches: 1-1634
Compiled by Arch Linux
Huge version without GUI.  Features included (+) or not (-):
+acl               -farsi             +mouse_sgr         +tag_binary
+arabic            +file_in_path      -mouse_sysmouse    -tag_old_static
+autocmd           +find_in_path      +mouse_urxvt       -tag_any_white
+autochdir         +float             +mouse_xterm       +tcl/dyn
-autoservername    +folding           +multi_byte        +termguicolors
-balloon_eval      -footer            +multi_lang        +terminal
+balloon_eval_term +fork()            -mzscheme          +terminfo
-browse            +gettext           +netbeans_intg     +termresponse
++builtin_terms    -hangul_input      +num64             +textobjects
+byte_offset       +iconv             +packages          +textprop
+channel           +insert_expand     +path_extra        +timers
+cindent           +ipv6              +perl/dyn          +title
-clientserver      +job               +persistent_undo   -toolbar
-clipboard         +jumplist          +popupwin          +user_commands
+cmdline_compl     +keymap            +postscript        +vartabs
+cmdline_hist      +lambda            +printer           +vertsplit
+cmdline_info      +langmap           +profile           +virtualedit
+comments          +libcall           +python/dyn        +visual
+conceal           +linebreak         +python3/dyn       +visualextra
+cryptv            +lispindent        +quickfix          +viminfo
+cscope            +listcmds          +reltime           +vreplace
+cursorbind        +localmap          +rightleft         +wildignore
+cursorshape       +lua/dyn           +ruby/dyn          +wildmenu
+dialog_con        +menu              +scrollbind        +windows
+diff              +mksession         +signs             +writebackup
+digraphs          +modify_fname      +smartindent       -X11
-dnd               +mouse             -sound             -xfontset
-ebcdic            -mouseshape        +spell             -xim
+emacs_tags        +mouse_dec         +startuptime       -xpm
+eval              +mouse_gpm         +statusline        -xsmp
+ex_extra          -mouse_jsbterm     -sun_workshop      -xterm_clipboard
+extra_search      +mouse_netterm     +syntax            -xterm_save
[...]
```

On peut voir que pour mon cas, les fonctionnalités `+python/dyn` (Python 2) et `+python3/dyn` (Python 3) sont activées.

```
$ vim --version | grep python
+comments          +libcall           +python/dyn        +visual
+conceal           +linebreak         +python3/dyn       +visualextra
```

Sachant que `Vim` peut être compilé de quatre façons différentes :
1. Pas de support Python (`-python`, `-python3`)
2. Support de Python 2 seulement (`+python` ou `+python/dyn`, `-python3`)
3. Support de Python 3 seulement (`-python`, `+python3` ou `+python3/dyn`)
4. Support de Python 3 et 3 (`+python/dyn`, `+python3/dyn`)

Lorsque Python 2 et Python sont tous les deux supportés, ils doivent être chargés dynamiquement. Cela signifie que `Vim` va rechercher le fichier de bibliothèque partagée uniquement lorsque c'est nécessaire.  
<br>
Le souci, lorsque l'on fait cela sur des systèmes Linux/Unix et que l'on importe des symboles globaux, un crash se produit lorsque la seconde version de Python est utilisée. Donc, pour résoudre notre problème, il faut que les symboles globaux soient activés, mais avec une seule version de Python activé.

Et pour cela, nous allons devoir recompiler le paquet `Vim`.


# Rebuild d'un paquet

## Installation des logiciels nécessaires

Avant de commencer, il est nécessaire d'installer deux logiciels nécessaires : `base-devel` et `asp`.  
Le premier est un groupe de paquet contenant des utilitaires permettant de compiler des logiciels. Le second est un outils permettant de récupérer les fichiers sources d'un paquet Arch Linux. Nous aurons également besoin du paquet `makepkg`, mais ce dernier installé de base avec `pacman`. 

```
$ sudo pacman -S base-devel asp
```

## Téléchargement des fichiers sources

Grâce à l'outil `asp`, nous pouvons télécharger les sources de Vim :

```
$ asp checkout vim
```

Un répertoire `vim` est crée dans notre répertoire courant. Ce dernier contient deux sous-répertoires.
* `repos` : contiens ses propres sous-répertoires en fonction de l'architecture du système.
* `trunk` : contiens tous les fichiers de développement du paquet.

```
vim
├── repos
│   └── extra-x86_64
│       ├── archlinux.vim
│       ├── PKGBUILD
│       ├── vimdoc.hook
│       └── vimrc
└── trunk
    ├── archlinux.vim
    ├── PKGBUILD
    ├── vimdoc.hook
    └── vimrc
```

## Modification d'un fichier PKGBUILD

> Nous allons modifier le fichier **trunk/PKGBUILD**

Dans le fichier `PKGBUILD`, la fonction `build()` va nous intéresser.  
Il faut modifier ce script de compilation en **supprimant** l'option `--enable-pythoninterp=yes` pour ne pas activer le support Python 2.

```
msg2 "Building vim..."
(cd vim-${pkgver}
  ./configure \
    --prefix=/usr \
    --localstatedir=/var/lib/vim \
    --with-features=huge \
    --with-compiledby='Arch Linux' \
    --enable-gpm \
    --enable-acl \
    --with-x=no \
    --disable-gui \
    --enable-multibyte \
    --enable-cscope \
    --enable-netbeans \
    --enable-perlinterp=dynamic \
    --enable-pythoninterp=yes \           <------
    --enable-python3interp=dynamic \
    --enable-rubyinterp=dynamic \
    --enable-luainterp=dynamic \
    --enable-tclinterp=dynamic \
    --disable-canberra
  make
)
```

De même pour la variable `makedepends`, nous pouvons **supprimer** le paquet `python2` des dépendances, car nous n'en aurons plus besoin.

```
makedepends=('glibc' 'libgcrypt' 'gpm' 'python2' 'python' 'ruby' 'libxt' 'gtk3' 'lua' 'gawk' 'tcl' 'pcre' 'zlib' 'libffi' 'libcanberra')
```

## Build d'un paquet

Avant de build le paquet, il faut récupérer la clé gpg de la personne ou du groupe s'occupant de build le paquet dans notre keyring. Cette étape est nécessaire, car la signature des fichiers source téléchargés pour le logiciel que nous voulons build est automatiquement vérifiée par rapport à une clé gpg, nous devons la posséder dans notre keyring, sinon le build échouera.

Pour cela, il faut récupérer des informations sur le paquet avec la commande suivante et c'est le champ `Packager` qui nous intéresse :

```
$ pacman -Qi vim
...
Packager        : Felix Yan <felixonmars@archlinux.org>
...
```

Je me doute qu'une méthode plus simple existe, mais je n'avais pas envie de me prendre la tête, donc j'ai simplement fais une recherche de `Felix Yan`, que l'on retrouve sur la [page des développeurs d'Arch Linux](https://www.archlinux.org/people/developers/#felixonmars), avec un lien fournissant sa clé gpg (`B5971F2C5C10A9A08C60030F786C63F330D7CB92`).

Nous pouvons donc récupérer cette clé gpg avec la commande suivante :

```
$ gpg --keyserver keys.openpgp.org --recv B5971F2C5C10A9A08C60030F786C63F330D7CB92
gpg: key 786C63F330D7CB92: no user ID
gpg: Total number processed: 1
```

Nous sommes maintenant prêts à build notre paquet. Pour cela, il faut se placer dans le répertoire contenant le fichier `PKGBUILD` que nous avons modifié précédemment et exécuter la commande suivante :

```
$ makepkg --clean --syncdeps --rmdeps
```

Rapide descriptif des options utilisées avec `makepkg` :
* `--clean, -c` : nettoie les fichiers et les répertoires restants après une compilation réussie
* `--syncdeps, -s` : installe les dépendances manquantes en utilisant pacman
* `--rmdeps, -r` : une fois la compilation réussie, supprime toutes les dépendances installées par makegpg lors de la résolution automatique des dépendances et de l'installation de l'utilisation de -s 

```
==> Making package: vim 8.2.1634-1 (Sun 13 Sep 2020 02:11:47 PM CEST)
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> Retrieving sources...
  -> Downloading vim-8.2.1634.tar.gz...
  -> Found vimrc
  -> Found archlinux.vim
  -> Found vimdoc.hook
==> Validating source files with sha256sums...
    vim-8.2.1634.tar.gz ... Passed
    vimrc ... Passed
    archlinux.vim ... Passed
    vimdoc.hook ... Passed
==> Validating source files with sha512sums...
    vim-8.2.1634.tar.gz ... Passed
    vimrc ... Passed
    archlinux.vim ... Passed
    vimdoc.hook ... Passed
==> Extracting sources...
  -> Extracting vim-8.2.1634.tar.gz with bsdtar
==> Starting prepare()...
==> Starting build()...
  -> Building vim...
[...]
```

Une fois que le build s'est terminé sans erreur, nous trouvons quatre nouveaux fichiers qui sont les paquets issus de la compilation :
```
gvim-8.2.1634-1-x86_64.pkg.tar.zst
vim-8.2.1634-1-x86_64.pkg.tar.zst
vim-8.2.1634.tar.gz
vim-runtime-8.2.1634-1-x86_64.pkg.tar.zst
```

Nous pouvons installer les paquets `vim` et `vim-runtime` avec `pacman` :

```
$ sudo pacman -U vim-8.2.1634-1-x86_64.pkg.tar.zst vim-runtime-8.2.1634-1-x86_64.pkg.tar.zst
```

# Conclusion

Dans cet article, nous avons vu comment identifier les différents modules supportés par une installation de `Vim`, mais également, comment télécharger les fichiers sources d'un paquet, comment modifier un fichier `PKGBUILD` et comment reconstruire le paquet par le biais de l'utilitaire `makepkg`.
<br>
D'une manière plus générale, nous avons vu comment il est possible d'utiliser `Arch Build System` pour re-build un paquet existant pour le modifier et l'adapter à nos besoins.

# Sources

Vous trouverez ci-dessous les différentes ressources qui m'ont permis de rédiger cet article.
* [https://github.com/ycm-core/YouCompleteMe/issues/3635](https://github.com/ycm-core/YouCompleteMe/issues/3635)
* [https://wiki.archlinux.org/index.php/Arch_Build_System](https://wiki.archlinux.org/index.php/Arch_Build_System)
* [https://wiki.archlinux.org/index.php/PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD)
* [https://wiki.archlinux.org/index.php/Makepkg](https://wiki.archlinux.org/index.php/Makepkg)
* [https://linuxconfig.org/how-to-rebuild-a-package-using-the-arch-linux-build-system](https://linuxconfig.org/how-to-rebuild-a-package-using-the-arch-linux-build-system)
* [https://vimhelp.org/if_pyth.txt.html#python-dynamic](https://vimhelp.org/if_pyth.txt.html#python-dynamic)
