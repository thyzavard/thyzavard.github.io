---
title: Capture.The.FIC 2019 - Forensic - Do you forensic ANALyst job
excerpt: Step 3 des pré-qualifications du Capture the FIC
tags: [CTF, Capture.The.FIC, forensic, writeup]
categories: Write-ups
---

![Challenge](/img/do_your_forensic_analyst_job_chall.png){: .center-image}


On nous fournis un fichier .raw qui est apparememnt le dump d'une clé USB:
```
$ file image.raw   
image.raw: DOS/MBR boot sector MS-MBR Windows 7 english at offset 0x163 "Invalid partition table" at offset 0x17b "Error loading operating system" at offset 0x19a "Missing operating system", disk signature 0x47e6da9e; partition 1 : ID=0x7, start-CHS (0x0,2,3), end-CHS (0xc5,3,19), startsector 128, 198656 sectors
```
En essayant de simplement monter l'image, on obtient une erreur. Via `fdisk`, on récupère les informations nécessaires pour ajuster l'option `offset` de la commande `mount`:
```
$ fdisk -l image.raw
Disk image.raw: 100 MiB, 104857600 bytes, 204800 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x47e6da9e

Device     Boot Start    End Sectors Size Id Type
image.raw1        128 198783  198656  97M  7 HPFS/NTFS/exFAT
```

On va donc essayer de monter la partition:
```
$ sudo mount -o loop,offset=65536 image.raw /mnt/t
mount: /mnt/t: unknown filesystem type 'BitLocker'.
```
Visiblement, il s'agit d'une partition Bitlocker. Pour s'en assurer, on peut vérifier si la signature "**-FVE-FS-**", correspondant à une partition Bitlocker chiffré est présente dans l'en-tête de l'image:
```
$ hexdump -C -s 65536 -n 16 image.raw
00010000  eb 58 90 2d 46 56 45 2d  46 53 2d 00 02 08 00 00  |.X.-FVE-FS-.....|
```
Un autre moyen de vérifier est d'ouvrir l'image avec Gparted:
![Challenge](/img/do_your_forensic_analyst_job_gparted.png){: .center-image}

Pour plus de simplicité pour les étapes suivantes, on extrait la partition Bitlocker de l'image:
```
$ dd if=image.raw bs=512 skip=128 count=198656 of=bitlocker.img 
```

Nous allons avoir besoin d'un outil permettant de monter des partitions Bitlocker. Après quelques recherches, nous avons trouvé l'outil [Dislocker](https://github.com/Aorimn/dislocker).<br>

Avant de monter la partition, il faut trouver le mot de passe ou la clé permettant de déchiffrer cette partition.
Après avoir lu quelques writeups de challenges similaire, nous avons essayé un outil permettant d'effectuer une attaque par dictionnaire sur une partition [Bitlocker](https://en.wikipedia.org/wiki/BitLocker).
La combinaison de [Dislocker-dict](https://wreckedsecurity.com/encryption-and-data-protection/brute-force-dictionary-attack-against-bitlocker/) avec la wordlist `rockyou` nous permet de trouver le mot de passe: `password`:
```
$ ./dislocker-dict -v output.img -m /mnt/bitlocker -d rockyou.txt
dislocker-dict v. 0.1 - Very inefficient way of using dictionary attack against BitLocker
(C) 2017 Wrecked Security
Running PASS 1
123456	 NOT FOUND
12345	 NOT FOUND
123456789	 NOT FOUND
password	 FOUND!
Partition decryption password is: password
```
On monte la partition avec `Dislocker` en fournissant le mot de passe. Et on obtient un fichier `dislocker-file`:
```
$ dislocker -V output.img -upassword /mnt/bitlocker
$ ls /mnt/bitlocker/
dislocker-file
```
On monte ce fameux fichier `dislocker-file` pour avoir accès au système de fichier de la clé USB:
```
$ sudo mount /mnt/bitlocker/dislocker-file /mnt/dislock
```
On trouve deux fichiers, dont un qui semble contenir le flag:
```
$ ls /mnt/dislock                  
flag.txt  'System Volume Information'
$ cat flag.txt
Every Forensic investigation starts with a good bitlocker inspection.
-- @chaignc

Try Harder !
```
:thumbsup::thumbsup::thumbsup::thumbsup::thumbsup:<br/>
Nous avons essayé de chercher si le répertoire `System Volume Information` pouvait nous être utile à la résolution de ce challenge. Sans succès.<br/><br/>

On utilise ensuite la commande `fls` afin de lister tous les fichiers présents dans la partition, y compris les fichiers supprimés que l'on repère grâce au caractère `*` : 
```
$ fls -r /mnt/bitlocker/dislocker-file
r/r 4-128-4:	$AttrDef
r/r 8-128-2:	$BadClus
r/r 8-128-1:	$BadClus:$Bad
r/r 6-128-4:	$Bitmap
r/r 7-128-1:	$Boot
d/d 11-144-4:	$Extend
+ r/r 25-144-2:	$ObjId:$O
+ r/r 24-144-3:	$Quota:$O
+ r/r 24-144-2:	$Quota:$Q
+ r/r 26-144-2:	$Reparse:$R
+ d/d 27-144-2:	$RmMetadata
++ r/r 28-128-4:	$Repair
++ r/r 28-128-2:	$Repair:$Config
++ d/d 30-144-2:	$Txf
++ d/d 29-144-5:	$TxfLog
+++ r/r 31-128-2:	$Tops
+++ r/r 31-128-4:	$Tops:$T
+++ r/r 32-128-1:	$TxfLog.blf
+++ r/r 33-128-1:	$TxfLogContainer00000000000000000001
+++ r/r 34-128-1:	$TxfLogContainer00000000000000000002
r/r 2-128-1:	$LogFile
r/r 0-128-1:	$MFT
r/r 1-128-1:	$MFTMirr
r/r 9-128-8:	$Secure:$SDS
r/r 9-144-11:	$Secure:$SDH
r/r 9-144-5:	$Secure:$SII
r/r 10-128-1:	$UpCase
r/r 3-128-3:	$Volume
r/r 39-128-1:	flag.txt
d/d 35-144-6:	System Volume Information
+ r/r 40-128-1:	FVE2.{24e6f0ae-6a00-4f73-984b-75ce9942852d}
+ r/r 36-128-1:	FVE2.{e40ad34d-dae9-4bc7-95bd-b16218c10f72}.1
+ r/r 37-128-1:	FVE2.{e40ad34d-dae9-4bc7-95bd-b16218c10f72}.2
+ r/r 38-128-1:	FVE2.{e40ad34d-dae9-4bc7-95bd-b16218c10f72}.3
+ -/r * 41-128-1:	FVE2.{c9ca54a3-6983-46b7-8684-a7e5e23499e3}
-/r * 64-128-2:	ls
-/r * 65-128-2:	fic.zip
-/r * 66-128-2:	f1.zip
-/r * 67-128-2:	f2.zip
-/r * 68-128-2:	f3.zip
-/r * 69-128-2:	f4.zip
V/V 256:	$OrphanFiles
```
Avec `icat`, on peut récupérer un fichier supprimé en fournissant un numéro d'inode: `65-158-2` pour le fichier `fic.zip`:
```
$ icat -r /mnt/bitlocker/dislocker-file 65-158-2 > fic.zip
```
Une autre solution consiste à utiliser `foremost` pour récupérer tous les fichiers ayant été supprimés dans la partition:
```
$ foremost -T -i /mnt/bitlocker/dislocker-file -o recovery -t all
$ tree recovery
.
├── audit.txt
└── zip
    ├── 00066346.zip
    ├── 00066348.zip
    ├── 00066350.zip
    ├── 00066352.zip
    ├── 00066354.zip
    └── fic.txt

1 directory, 7 files
```
On va simplement extraire l'archive et lire le contenu du fichier extrait:
```
$  unzip 00066346.zip 
Archive:  00066346.zip
  inflating: fic.txt                 
```
Il y a un lien Github Gist:
```
$ cat fic.txt 
https://gist.github.com/bosal43833/3e815abc3f92e45963a8aafc8acfe411
```
Qui contient le flag en base64:
![Gist](/img/do_your_forensic_analyst_job_gist.png){: .center-image}
:checkered_flag:FLAG:checkered_flag::
```
$ echo aHR0cHM6Ly9jdGYuaGV4cHJlc3NvLmZyLzFlYTk2N2Y1MmQxYWFiMzI3ZDA4NGVmZDI0ZDA0OTU3Cg== | base64 --decode
https://ctf.hexpresso.fr/1ea967f52d1aab327d084efd24d04957
```

