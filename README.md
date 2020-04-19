# PVKuvahaaste2020 

On 17.4 the Finnish Defence Forces tweeted this [challenge](https://twitter.com/Puolustusvoimat/status/1251139182111739904)
[TODO](https://asdasd) decided to take a a swing at the challenge and he quickly revealed that the data had a link to mysterious *.bin*
 file. 
 
## The barcode

Simply reading the barcode and encoding the data first to ```base32``` and then ```rot47``` revealed a link to a site where you could download a mysterious bin file. 
-etc

## The decoy

I started getting interested when L had found this address ```106.250.84.114:82``` which didin't point to anything with a regular web browser. After some searching we found [this](https://www.speedguide.net/port.php?port=82) that specified that this particular port had been used for.

* some kind of tor tool
* xfer
* 10 years old Win32 Trojan

We first tried to visit the page with a Tor browser without any response. The aspect of Tor makes it really hard to trace, so Wireshark inspection was really tedious so we gave up.

#### nmap

A quick nmap scan reveladed that server didn't have much running with TCP. Only the port 21 was interesting. Connecting to it with anonymous ftp obviously didn't work at all. 
```
Host is up (0.35s latency).
Not shown: 994 closed ports
PORT    STATE    SERVICE
21/tcp  open     ftp
23/tcp  filtered telnet
25/tcp  filtered smtp
135/tcp filtered msrpc
139/tcp filtered netbios-ssn
445/tcp filtered microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 49.97 seconds****
```
The reason was quite obvious, a deeper scan for the service banners reveled that the ```port 21``` *may* have had some kind of firewall running or nmap didin't understand what it was talking to. Inspecting the TCP protocol exchange more deeply didin't reveal anything either. The server just disconnected after the first handshake attempt, so even sending packets with Netcat wouldn't work.

```
21/tcp  open     tcpwrapped
```

The original ```port 82``` didin't have anything interesting running either.
```
Host is up (0.34s latency).

PORT   STATE  SERVICE
82/tcp closed xfer

Nmap done: 1 IP address (1 host up) scanned in 1.81 second
```
I experimented with the this [xfer](https://docs.ipswitch.com/MOVEit/DMZ90/FreelyXfer/MOVEitXferManual.html) utility, but a closer inspection at the source code revealed that it was just a fancy file transfer wrapper that was really old. Xfer clients seemed to have disappeared from the face of the earth around 10 years ago.

#### Trojan thoughts

I wasted a lot of time digging through what xfer was and finding any tools that could use said protocol without any luck. I came to the conclusion that if this was the correct server then we would need some additional information or we would need to exploit some kind of old vurnerability which the port 82 reference could point to. I could have written a small .exe and try to send it via netcat to get the server ping to one of my PfSense firewalls. 

In the end my theory was proven to be very false or then PV really is playing some 4D chess.


## The real IP

TODO

We landed on the real page that look something like this:

![image](./damn.png)

From know on you can guess that challenge was almost done.

### Python script to solve the puzzle

You can view the abstracted Python scipt [here](./solver.py)

```python
import requests
from bs4 import BeautifulSoup

orig_url = "http://104.xx.xx.xx"
page = BeautifulSoup(requests.get(orig_url).text, 'lxml')
key = page.p.findAll('b')[0].text
#url = page.p.findAll('b')[1].text

secrets = ["AAAAAAA","AAAAAAA","AAAAAAA","AAAAAAA","AAAAAAA","AAAAAAA",]

for s in secrets:
    payload = {
        'secret': s,
        'key': key
    }
    print(requests.request("POST", orig_url, headers=headers, data = payload).text)  
```

First we pull the secret and the possible another ip address from the original page with ```GET``` and then iterate through the found secrets with the given instructions. So that would be ```POST``` with the secret, key header values. 

After that we got the print:

```bash
python3 solver.py
CORRECT! This is the end of the challenge. Write this string -> BRUH <- down, we might ask it.

ERROR

ERROR

ERROR

ERROR

ERROR
```

The ```ERROR``` prints just tell that there was ony one correct key for the post URL in the whole binary.
