#  Wi-Fight A Ghost? 
### Category : Forensics

<img width="475" height="529" alt="image9" src="https://github.com/user-attachments/assets/4728ff95-01cb-4a38-84be-898f5fc4ff1e" />

Voici toutes les questions auxquels nous devons répondre pour ce challenge : 

![alt](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/image_questions.png)

Pour ce faire nous avons une capture du drive `C:` sur Windows, effectué avec KAPE. 
Pour le début de l'investigation, j'utilise l'outil d'analyse **Autopsy** 

![alt](https://github.com/CreaTikaa/CTFs-Writeups/blob/main/L3ak%20CTF%202025/screenshots/autopsy.png)

**Question 1 : What was the ComputerName of the device ?** 
Je vais dans `Operating System Information` sur Autopsy pour avoir le nom de l'ordinateur : 

![computer_name](https://github.com/user-attachments/assets/1af14e0a-b046-4376-9cc6-fe166c825415)

Réponse : 99PHOENIXDOWNS

**Question 2 : What was the SSID of the first Wi-Fi network they connected to ?**
Pour trouver les différents SSID sur l'ordinateur, je vais voir la liste des interfaces enregistrés dans `C\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces{18C11DBD-93AB-4CA9-A804-4F} : 
<img width="1734" height="661" alt="xml1" src="https://github.com/user-attachments/assets/94e771e4-0306-4bae-8efe-1db58f91f889" />

Réponse : mugs_guest_5G

**Question 3 : When did they obtain the DHCP lease at the first café ?**

Pour avoir la date exact du premier lease DHCP, il suffit d'aller voir dans les logs d'évènements `Microsoft-Windows-DHCP-Client/Admin.evtx` et puis de trier l'ID d'évènement `50067` par date, ce qui nous donne : 

<img width="906" height="642" alt="dhcp_lease2" src="https://github.com/user-attachments/assets/dcbd594f-938d-48f3-aa42-940cfb1d664e" />

Puis on la converti en UTC pour correspondre au format de la réponse précicé dans l'énoncé du challenge
<img width="661" height="145" alt="dhcputc" src="https://github.com/user-attachments/assets/caf3289c-46d3-4449-ac81-5c627ce02ec2" />

Réponse : `2025-05-14 00:13:36`

**Question 4 : What IP address was assigned at the first café ?**
Pour obtenir cette information, on va faire un tour dans le **Registry Explorer** de Eric Zimmerman (merci à lui). On va voir au path `Microsoft\Windows NT\CurrentVersion\NetworkList` et on y trouve toute ses informations :

![interfaces_infos](https://github.com/user-attachments/assets/928ff959-f718-4a7e-acd9-f12ce17a37a9)

Réponse : 192.168.0.114

**Question 5 : What GitHub page did they visit at the first café ?**

Pour cela, je me suis rendu au path C:\Users\NotVi\AppData\Local\Google\CHrome\User Data\Default, la ou l'historique et d'autres informations sont présentes, et j'ai ouvert la database contenant l'historique avec **DB Browser** : 
Puis avec une simple query SQL, on voit tout les URLs visités, on sélectionne celui sur GitHub : 

![bluebook](https://github.com/user-attachments/assets/67addc8c-1292-4912-90c1-ded0f701a67f)

Réponse : https://github.com/dbissell116/DFIR/blob/main/Blue_Book/Blue_Book.md

**Question 6 : What did they download at the first café ?**

Après avoir fouillé l'historique des téléchargements sur Chrome, je ne trouve rien. Je décide donc d'aller voir du coté de Microsoft Edge au path `C\Users\NotVi\AppData\Local\Microsoft\Edge\User Data\Default`. En fouillant l'historique, je vois qu'il l'a utilisé pour installer Chrome, je vais donc voir l'historique des téléchargements : 

<img width="1856" height="132" alt="quest6" src="https://github.com/user-attachments/assets/34fd025d-8b04-4424-a260-b8f8725a0fb6" />

Réponse : ChromeSetup.exe

**Question 7 : What was the name of the notes file ?**

J'avais déjà remarqué celui la en me balladant dans le disque au début du challenge. En effet, dans `C\Users\NotVi\AppData\Roaming\Microsoft\Windows\Recent` On remarque ce fichier, qui attrape facilement le regard en plus : 

<img width="685" height="256" alt="quest7" src="https://github.com/user-attachments/assets/572c2410-c25d-44ce-8041-c293605bd3df" />

Réponse : HowToHackTheWorld.txt

**Question 8 : What are the contents of the notes ?**

Celui la m'a donné du mal. Beaucoup de mal. Après avoir cherché un peu partout, dans différents caches, dans `Windows.db` ou encore dans des endroits plus farfelues comme l'historique des commandes PowerShell, j'ai utilisé l'outil **MFT Explorer** de Eric Zimmerman (encore) pour fouiller le `$MFT`.
Et c'est dedans, que dans `C\Users\NotVi\OneDrive\Desktop` que j'avais noté via les propriétés du fichier HowToHackTheWorld.txt dans `..\Recent` que je trouve enfin la réponse que je recherche :

<img width="1917" height="1032" alt="8 mft" src="https://github.com/user-attachments/assets/6c8cefad-aeba-4425-ab81-333f771e8bbe" />

Réponse : Practice and take good notes.

**Question 9 : What are the SSID of the second Wi-Fi network they connected to ?**

On peut déjà répondre à cette question avec nos informations de la question 2, mais si on veut être sur, on peut aller vérifier dans `Microsoft-Windows-WLAN-AutoConfig/Operational.evtx ` puis trier les connexions par ordre chronologique, pour voir qu'il se connecte à `AlleyCat` après `mugs_guest_5G`.

Réponse : AlleyCat

**Question 10 : When did they obtain the second lease ?**
Pour cette question, on cherche exactement de la même manière que pour la question 3.

<img width="1189" height="645" alt="dhcp_lease1" src="https://github.com/user-attachments/assets/058a6811-678e-4f61-a9ba-7422f76dac57" />

Puis on convertit aussi en UTC, et on trouve la réponse.

Réponse : `2025-05-14 00:35:07`

**Question 11 : What was the IP address assigned at the second café ?**
Encore une fois, nous avons déjà la réponse à cette question grâce aux informations obtenues auparavant pour la question 4. 
Donc il suffit de reprendre notre capture d'écran et de noter l'autre adresse IP.

![interfaces_infos](https://github.com/user-attachments/assets/3c099d70-ea7b-44ab-a50f-5ee4f0b0ef33)

Réponse : 10.0.6.28

**Question 12 : What website did they log into at the second café ?**

Retour dans **DB Browser** pour aller visiter l'historique Chrome comme déjà fait auparavant.

<img width="820" height="292" alt="website leak" src="https://github.com/user-attachments/assets/7737816e-cea3-403c-9557-2fce8746c87d" />

Réponse : l3ak.team

**Question 13 : What was the MAC address of the Wi-Fi adapter used ?**

On se rend à nouveau dans `Microsoft-Windows-WLAN-AutoConfig/Operational.evtx` pour trouver un évènement `11004` qui contient le SSID réseau `AlleyCat` recerché, et la réponse.

<img width="539" height="228" alt="mac_address" src="https://github.com/user-attachments/assets/d374b2c5-7370-46ad-b17e-5f7ae2415dec" />

Réponse : `48:51:C5:35:EA:53`
(et la réponse était aussi au même endroit que les IPs des deux interfaces, mais j'ai répondu à cette question avant les autres)

**Question 14 : What city did this take place in ?**

Et pour finir, on trouve ou ce cache ce fameux **Ghost**, non sans un peu de mal dans mes recherches, mais au bout d'un moment on tombe bien sur ceci : 

![localisation](https://github.com/user-attachments/assets/515a8ade-f9f3-4978-b662-bca4f66130cd)

Réponse : Fort Collins 

**Conclusion**

Une fois toutes les questions répondus, on obtient enfin le flag final : `L3AK{Gh057_R!d!ng_7h3_W4v35}`

