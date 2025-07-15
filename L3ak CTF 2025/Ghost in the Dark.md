# Ghost in the Dark
### Category : Forensics

[image 1]

Dès qu'on récupère ce fichier, on vérifie son type. 

[image 2]

Une fois son type connu, je sais que je peux utiliser `tsk_recover` pour voir si il est possible de récupérer des choses à l'intérieur. Et en effet, on trouve le script `loader.ps1`.

[ image 3]

Avec ce script, on peut déchiffrer le fichier `payload.enc`, encore faut t'il le trouver, pour ça je vais lister tout les fichiers avec `fls` : 

[ image 4]

On remarque ici 2 fichiers particulièrement intéréssant, **payload.enc** et **flag.enc** (ransom_note.txt en bonus pour voir ce que le créateur du challenge nous as laissé comme beau message)

[ image 5}

Une fois qu'on a récupéré `payload.enc`, on peut donc le déchiffrer grâce au code de `loader.ps1`.

[ image 6}

En récupérant payload.enc en cleartext, on peut y lire `ransom_note.txt` mais aussi voir comment il encrypte le fichier **flag.txt** pour le faire devenir `flag.enc`. On suit donc le même processus que de loader.ps1 à payload.enc, en changeant juste la clé AES et l'IV dans le code.

[ image 7 ]

Ce qui nous donne, ce beau petit flag !

[ flag]
