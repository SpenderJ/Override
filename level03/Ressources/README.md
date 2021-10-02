# Override
Override 42 security level 3

```
objdump -M intel -d level03
```

On observe 3 fonctions, main test et decrypt.
Le main appelle la fonction test, qui elle même appelle decrypt.
On observe que le main genere un nombre aleatoire basé sur le temps actuel.
Un int est lu sur l'entrée standard et passé à la fonction test ainsi qu'un nombre en dur:
```
 80488ca:	c7 44 24 04 0d d0 37 	mov    DWORD PTR [esp+0x4],0x1337d00d -> 0x1337d00d
```
La fonction test soustrait le nombre passé en argument au nombre en dur.
Les deux nombres sont positifs (ja).
Afin d'appeler decrypt, il faut que cette difference soit superieur à 0x15:
```
 804875c:	83 7d f4 15          	cmp    DWORD PTR [ebp-0xc],0x15
 8048760:	0f 87 e4 00 00 00    	ja     804884a <test+0x103>
```

```
0x1337d00d -> 322424845
0x15 -> 21

# Donc:
# Afin de ne pas rentrer dans le random il faut que
322424845 - 21 = 322424824
# Donc notre nombre sera compris entre 322424824 et 322424845.
# Ne s'agissant que de 21 possibilitées nous allons les tester une à une afin de résoudre notre problème.
```

```
for x in range {322424824..322424845}; do (python -c  "print $x"; cat -) | ./level03 ; done
***********************************
*		level03		**
***********************************
Password:
Invalid Password

***********************************
*		level03		**
***********************************
Password:
Invalid Password

***********************************
*		level03		**
***********************************
Password:
Invalid Password

***********************************
*		level03		**
***********************************
Password:
Invalid Password

***********************************
*		level03		**
***********************************

whoami
level04
cat /home/users/level04/.pass
kgv3tkEb9h2mLkRsPkXRfc2mHbjMxQzvb2FrgKkf
```
