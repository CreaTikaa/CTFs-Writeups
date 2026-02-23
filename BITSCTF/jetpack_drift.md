# Write-up CTF : Jetpack drift

## Introduction
Dans ce challenge, nous incarnons un enquêteur qui doit récupérer un fichier téléchargé lors d'un échange réseau intercepté. Le téléchargement a "échoué", et le serveur est décrit comme "cassé". Nous avons à notre disposition une capture réseau (`chall.pcap`).

---

## Étape 1 : Analyse du mécanisme de chiffrement
Dans le chall.pcap, je remarque du traffic HTTP donc j'extrait les objets HTTP, ce qui nous donne : 
1. Le fichier `database.sql` contenant 250 utilisateurs avec leurs adresses email et leurs mots de passe en clair. L'un de ces mots de passe a été utilisé pour chiffrer le fichier.
2. Le fichier `encryption.py` nous dévoile la logique derrière le chiffrement :
   - Le fichier est découpé en chunks de 12 800 octets.
   - Le chiffrement utilisé est AES-CTR.
   - **Clé du 1er bloc :** Le hash SHA-256 du mot de passe de l'utilisateur.
   - **Clés suivantes :** Le hash SHA-256 du plaintext du chunk précédent.
   - **Structure en liste chaînée :** Chaque bloc est précédé d'un en-tête `NXTCHNKHASH:<hash>DATA:` où `<hash>` est le hash du chunk chiffré suivant. Le tout dernier chunk pointe vers un hash composé uniquement de zéros (`000...000`).

## Étape 2 : Extraction des données 
Ayant utilisé la fonction "Export HTTP Objects" de Wireshark, je me suis vite rendu compte que je n'allais pas avancer comme ça, le serveur web utilise l'en-tête `Transfer-Encoding: chunked`. Ce qui signifie que le protocole HTTP injecte des tailles de blocs en hexadécimal (ex: `1bcb\r\n`) directement au milieu du payload chiffré. L'AES-CTR étant extrêmement sensible à l'alignement, ces octets parasites corrompent totalement le déchiffrement.

De plus, l'indice "serveur cassé" vient nous mettre en garde de la chose suivante : le serveur a envoyé la liste chaînée de blocs à l'envers (le dernier bloc, pointant vers `000...000`, arrive en premier).
Donc pour régler ce problème, dans Wireshark, nous filtrons sur la requête HTTP contenant les données, on Follow TCP Stream puis on save le flux complet en RAW dans `raw_stream.bin`.


## Étape 3 : Le script de résolution final
Maintenant qu'on à tout les éléments qu'il nous fait, on va faire un script python pour chopper le flag : 
1. Lire le flux TCP brut et nettoyer l'encodage "HTTP chunked" pour récupérer un texte chiffré pur.
2. Séparer les blocs grâce aux balises `NXTCHNKHASH:` et reconstruire la liste chaînée dans le bon ordre.
3. Bruteforcer le premier bloc avec les 250 mots de passe de `database.sql`. Pour vérifier si le mot de passe est le bon sans connaître le type de fichier, on utilise l'entropie (donnes bien déchiffrés se compressent bien contrairement au bruit random que gen une autre clé)
4. Déchiffrer le reste du fichier en cascade.

Voici le script fonctionnel (`solve.py`) :

```python
import os
import hashlib
import re
import zlib
from Crypto.Cipher import AES
from Crypto.Util import Counter

# --- 1. Récupération des mots de passe ---
passwords = []
with open("database.sql", "r", encoding="utf-8", errors="ignore") as f:
    matches = re.findall(r"\('.*?', '.*?', '(.*?)'\)", f.read())
    passwords.extend(matches)

def sha256(data): return hashlib.sha256(data).digest()
def sha256_hex(data): return hashlib.sha256(data).hexdigest()

def decrypt_ctr(data, key):
    nonce = key[:16]
    counter = Counter.new(128, initial_value=int.from_bytes(nonce, 'big'))
    cipher = AES.new(key, AES.MODE_CTR, counter=counter)
    return cipher.decrypt(data)

# --- 2. Nettoyage de l'encodage HTTP Chunked ---
with open("raw_stream.bin", "rb") as f:
    stream_data = f.read()

response_start = stream_data.find(b"HTTP/1.1 200 OK")
headers_end = stream_data.find(b"\r\n\r\n", response_start)
body = stream_data[headers_end + 4:]

unchunked_data = bytearray()
idx = 0
while idx < len(body):
    nl_idx = body.find(b"\r\n", idx)
    if nl_idx == -1: break
    try:
        chunk_size = int(body[idx:nl_idx].strip().decode('ascii'), 16)
    except ValueError:
        break
    if chunk_size == 0: break
    
    chunk_start = nl_idx + 2
    unchunked_data.extend(body[chunk_start : chunk_start + chunk_size])
    idx = chunk_start + chunk_size + 2

# --- 3. Découpage et Réorganisation des blocs ---
chunks_info = {}
all_next_hashes = set()

parts = unchunked_data.split(b"NXTCHNKHASH:")
for part in parts:
    if not part: continue
    data_idx = part.find(b"DATA:")
    if data_idx == -1: continue
    
    next_hash = part[:data_idx].decode('ascii')
    enc_data = bytes(part[data_idx + 5:])
    
    curr_hash = sha256_hex(enc_data)
    chunks_info[curr_hash] = {
        "next_hash": next_hash,
        "enc_data": enc_data
    }
    all_next_hashes.add(next_hash)

# Trouver le bloc n°0 (celui pointé par aucun autre hash)
first_hash = None
for curr_hash in chunks_info.keys():
    if curr_hash not in all_next_hashes:
        first_hash = curr_hash
        break

ordered_enc_chunks = []
curr = first_hash
while curr in chunks_info:
    ordered_enc_chunks.append(chunks_info[curr]["enc_data"])
    curr = chunks_info[curr]["next_hash"]
    if curr == "0"*64: break

# --- 4. Bruteforce intelligent sur le Bloc 0 (par Entropie) ---
results = []
enc_0 = ordered_enc_chunks[0]

for pwd in passwords:
    key = sha256(pwd.encode('utf-8'))
    pt = decrypt_ctr(enc_0, key)
    
    # La vraie donnée se compresse mieux que du bruit aléatoire
    ratio = len(zlib.compress(pt)) / len(pt)
    results.append({"password": pwd, "ratio": ratio, "plaintext": pt})

results.sort(key=lambda x: x["ratio"])
winner = results[0]

print(f"[+] Mot de passe trouvé : {winner['password']} (Ratio de compression: {winner['ratio']:.4f})")

# --- 5. Déchiffrement en cascade final ---
final_data = bytearray()
prev_key = sha256(winner['password'].encode('utf-8'))

for enc in ordered_enc_chunks:
    pt = decrypt_ctr(enc, prev_key)
    final_data.extend(pt)
    prev_key = sha256(pt) # La clé du suivant est le hash du plaintext actuel

with open("jetpack_drift_flag", "wb") as f:
    f.write(final_data)

print("[*] Fichier déchiffré sauvegardé sous 'jetpack_drift_flag' !")

```


## Étape 4 : Identification du fichier et récupération du flag

On obtient un fichier avec une entropyfaible (`~0.68`), le mot de passe est donc (`1VL7p6Rcli8mxgkh`).

On regarde quel type de fichier c'est, puis ses magic bytes pour bien l'identifier : 
```bash
$ file jetpack_drift_flag
jetpack_drift_flag: data
$ xxd jetpack_drift_flag | head -n 5
00000000: 0000 0020 6674 7970 6973 6f6d 0000 0200  ... ftypisom....

```

L'en-tête `ftypisom` pointe vers une video MP4.

Il ne reste plus qu'à renommer le fichier pour pouvoir le visionner :

```bash
$ mv jetpack_drift_flag jetpack_drift_flag.mp4

```

On lance la vidéo dans VLC, et on entend juste une voix avec un émoji qui rigole. La voix me semble étrange, comme si elle était mise à l'envers. Donc j'extrait l'audio, je le reverse, et la voix qui disait des trucs incompréhensible dit maintenant "Change the format to PNG". On avait donc affaire à un fichier "polyglot."  
Je change le format en PNG, et le flag est marqué en gros sur l'image.
