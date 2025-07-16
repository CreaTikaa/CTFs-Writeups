# Ghost in the Dark
### Category : Forensics

![image 0](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/image0.png)

Dès qu'on récupère ce fichier, on vérifie son type. 

![image 1](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/image1.png)
Une fois son type connu, je sais que je peux utiliser `tsk_recover` pour voir si il est possible de récupérer des choses à l'intérieur. Et en effet, on trouve le script `loader.ps1`.

![image 2](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/image2.png)

Avec ce script, on peut déchiffrer le fichier `payload.enc`, encore faut t'il le trouver, pour ça je vais lister tout les fichiers avec `fls` : 

![image 3](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/image3.png)

On remarque ici 2 fichiers particulièrement intéréssant, **payload.enc** et **flag.enc** (ransom_note.txt en bonus pour voir ce que le créateur du challenge nous as laissé comme beau message)

![image 4](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/image4.png)

Une fois qu'on a récupéré `payload.enc`, on peut donc le déchiffrer grâce au code de `loader.ps1`.

![image 5](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/image5.png)

En récupérant payload.enc en cleartext, on peut y lire `ransom_note.txt` mais aussi voir comment il encrypte le fichier **flag.txt** pour le faire devenir `flag.enc`. On suit donc le même processus que de loader.ps1 à payload.enc, en changeant juste la clé AES et l'IV dans le code.

![image 6](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/image6.png)

Voici le résultat, une fois déchiffré :

![image 7](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/image7.png)

Ce qui nous donne, ce beau petit flag !

![image 8](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/image8.png)
