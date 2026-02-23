# Challenge Write-up: Tuff Game (BITSCTF)


**Description :** Reach 1,000,000 points in the Unity game to retrieve the hidden flag.


## Step 1: Reco
On nous donne un jeu unity, pour comprendre son fonctionnement il faut analyser `Tuff_Game_Data/Managed/Assembly-CSharp.dll`.


## Step 2: Fakes flag (de merde)

### Fake 1: XOR 
Dans la classe `FlagGeneration`, une méthode `decryptFlag()` faisait un XOR sur un `encryptedFlag` avec comme clé `KEY = 0x5A`.
En déchiffrant ça, on tombe sur : `{Umm_4ctually_unx0r11ng_t0_g3t_fl4g_s33ms_t00_34sy}`.

### Fake 2: RSA Cryptography
Une autre classe qui s'appelle `NotAFlag` contennait 9 integers (`n1` à `n9`) et un `ciphertext`.
Factoriser `n6` pour déchiffrer le ciphertext donnait `BITSCTF{https://blogs.mtdv.me/Crypt0}`.  
Ce qui nous mène vers un rick roll (lol)


## Step 3: Game assets
Code inutile, on passe aux assets, quand on reach `1,000,000` distance, la méthode `ScoreManaged.ShowFlagAndRetry()` est call :

```csharp
public void ShowFlagAndRetry()
{
    Time.timeScale = 0f;
    this.flagImage.gameObject.SetActive(true);
    this.retryScreenUI.SetActive(true);
}
```

Donc le flag est une image (`flagImage`).

### Extraire l'asset :
On utilise **UnityPy** pr dump toutes les textures de `sharedassets0.assets` & `resources.assets`.

```python
import UnityPy
import os
from PIL import Image

env = UnityPy.load('sharedassets0.assets')
for obj in env.objects:
    if obj.type.name == 'Texture2D':
        try:
            data = obj.read()
            img = data.image
            img.save(f"{data.name}.png")
        except:
            pass
```

## Step 4: Flag framentée
Dans `sharedassets0.assets` on trouve 2 images plus grosses que les autres :   
La premiere contient `BITSCTF{début d'un flag}`  
Et la seconde `"Gotcha, perhaps think vertically"`.


## Step 5: Reconstruction of the QR Code
Dans resources.assets, mon copain Gemini à trouver 900 petits textures de 5x5 pixels avec des noms de coordonées : `rq_0_0`, `rq_0_1`, ..., `rq_29_29`. Ce qui forme une grid 30x30.

On assemble tout les petits blocs : 

```python
import os
from PIL import Image
import UnityPy

env = UnityPy.load("resources.assets")
blocks = {}
max_x = max_y = 0

for obj in env.objects:
    if obj.type.name == "Texture2D":
        try:
            data = obj.read()
            if data.name.startswith("rq_"):
                parts = data.name.split("_")
                x, y = int(parts[1]), int(parts[2])
                blocks[(x, y)] = data.image
                max_x, max_y = max(max_x, x), max(max_y, y)
        except:
            pass

assembled = Image.new("RGBA", (150, 150))
for (x, y), img in blocks.items():
    assembled.paste(img, (x * 5, y * 5))

assembled.save("flag_qr.png")
```

## Step 6: Flag
Ce qui nous donne un petit `flag_qr.png`, qui est un QRcode tout à fait valide, et qui une fois scanné, nous donne le flag.

**Flag:** `BITSCTF{Th1$_14_D3f1n1t3ly_Th3_fl4g}`
