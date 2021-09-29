# Override
Override 42 security lvl 00

```
gdb ./level00
disass main

0x080484e7 <+83>:	cmp    $0x149c,%eax
```
En traduisant le main on comprend que'il y a un scanf qui attend un input.
Cet input est comparé à la valeur 0x149c, qu'on traduit en décimal.
0x149c en decimal donne 5276.

```
./level00
5276
Authenticated!
$ cat /home/users/level01/.pass
uSq2ehEGT6c9S24zbshexZQBXUGrncxn5sD5QfGL
su level01
```
