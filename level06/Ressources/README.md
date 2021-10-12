# Override
Override 42 security level06

```
level06@OverRide:~$ ./level06
***********************************
*		level06		  *
***********************************
-> Enter Login: bonsoir
***********************************
***** NEW ACCOUNT DETECTED ********
***********************************
-> Enter Serial: 30071998
level06@OverRide:~$
```

En lisant l'assembleur, on remarque un fgets permettant de rentrer le login puis un scanf pour le nombre.
On doit faire valider la fonction auth pour reussir le niveau (return 0).
Le hack se passera en 2 etapes.
Comme on l'observe dans le source code, la fonction est protégée contre le debug.
On va donc devoir faire sauter l'appel à ptrace pour pouvoir regarder la valeur comparé.
On lance donc notre executable avec gdb, puis on met un breakpoint dans le main puis pour la fonction auth.
On se place aux emplacements des comparaisons, dans un premier temps pour remplacer ptrace, puis pour regarder la valeur nous interessant.
On relance l'executable avec le meme identifiants et la valeur nous interessant:
```
(gdb) b * main
Breakpoint 1 at 0x8048879
(gdb) r
Starting program: /home/users/level06/level06

Breakpoint 1, 0x08048879 in main ()
(gdb) b * auth
Breakpoint 2 at 0x8048748
(gdb) continue
Continuing.
***********************************
*		level06		  *
***********************************
-> Enter Login: test123
***********************************
***** NEW ACCOUNT DETECTED ********
***********************************
-> Enter Serial: 42

Breakpoint 2, 0x08048748 in auth ()
(gdb) disass
0x080487ba <+114>:	cmp    $0xffffffff,%eax
0x08048866 <+286>:	cmp    -0x10(%ebp),%eax

End of assembler dump.
(gdb) b *0x080487ba
Breakpoint 3 at 0x80487ba
(gdb) b *0x08048866
Breakpoint 4 at 0x8048866
(gdb) continue
Continuing.
[Inferior 1 (process 1874) exited with code 01]
(gdb) r
Starting program: /home/users/level06/level06

Breakpoint 1, 0x08048879 in main ()
(gdb) continue
Continuing.
***********************************
*		level06		  *
***********************************
-> Enter Login: test123
***********************************
***** NEW ACCOUNT DETECTED ********
***********************************
-> Enter Serial: 123

Breakpoint 2, 0x08048748 in auth ()
(gdb) continue
Continuing.

Breakpoint 3, 0x080487ba in auth ()
(gdb) set $eax=0
(gdb) continue
Continuing.

Breakpoint 4, 0x08048866 in auth ()
(gdb) x/wx $ebp-0x10
0xffffd238:	0x005f14a7 # en decimal 6231207
(gdb) quit
A debugging session is active.

	Inferior 1 [process 1877] will be killed.

Quit anyway? (y or n) y
level06@OverRide:~$ ./level06
***********************************
*		level06		  *
***********************************
-> Enter Login: test123
***********************************
***** NEW ACCOUNT DETECTED ********
***********************************
-> Enter Serial: 6231207
Authenticated!
$ cat /home/users/level07/.pass
GbcPDRgsFK77LNnnuh7QyFYA2942Gp8yKj9KrWD8
```
