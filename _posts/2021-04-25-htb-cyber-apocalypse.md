---
layout: post
title: "HTB Cyber Apocalypse CTF 2021"
date: 2021-04-25
---

A 5 crazy and intense days, the Cyber Apocalypse CTF 2021 ran from Monday, 19th of April 2021 until Friday, 23rd of April 2021. Flagged (excuse the pun) as a must-attend event for every cybersecurity enthusiast, my team Hack South saw two separate teams enter the races. Our particular team ended off the event placing a respectable `106 out of 4,700` of teams.  

Here is a [full event recap](https://www.hackthebox.eu/newsroom/cyber-apocalypse-ctf-2021-event-recap) from the organisers if you're interested.

---

## Challenges

I went into this CTF with a particular focus on the cryptography challenges. Why? I have no idea but I'm glad I did as the bug has bitten and I've been doing a lot of research in this space since the CTF.

1. [Nintendo Base64 [Cryptography]](#1)
1. [PhaseStream 1 [Cryptography]](#2)
1. [PhaseStream 2 [Cryptography]](#3)
1. [PhaseStream 3 [Cryptography]](#4)
1. [MiniSTRyplace [Web]](#5)

**Team write-ups:** [spymky](https://spymky.dev/ctf/cyber-apocalypse/){:target="_blank"}

---

## <a name="1"></a>Nintendo Base64
> Aliens are trying to cause great misery for the human race by using our own cryptographic technology to encrypt all our games. Fortunately, the aliens haven't played CryptoHack so they're making several noob mistakes. Therefore they've given us a chance to recover our games and find their flags. They've tried to scramble data on an N64 but don't seem to understand that encoding and ASCII art are not valid types of encryption!

1. This was a pretty straightforward challenge given the hint of `base64` in the title. I grabbed `output.txt` and ran it through a couple of base64 decodes in [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)From_Base64('A-Za-z0-9%2B/%3D',true)) until I eventually got the flag.

![](/assets/htbca2021/02.png)

1. It required 8 decodes which explains the 64x`8` when looking at the `output.txt` ASCII art.

![](/assets/htbca2021/01.png)

---

## <a name="2"></a>PhaseStream 1
> The aliens are trying to build a secure cipher to encrypt all our games called "PhaseStream". They've heard that stream ciphers are pretty good. The aliens have learned of the XOR operation which is used to encrypt a plaintext with a key. They believe that XOR using a repeated 5-byte key is enough to build a strong stream cipher. Such silly aliens! Here's a flag they encrypted this way earlier. Can you decrypt it (hint: what's the flag format?)

`2e313f2702184c5a0b1e321205550e03261b094d5c171f56011904`

1. Another easy challenge considering that we know the beginning of the flag format `CHTB{` and that the aliens have learned about `XOR`. In [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')Take_bytes(0,5,false)XOR(%7B'option':'UTF8','string':'CHTB%7B'%7D,'Standard',false)&input=MmUzMTNmMjcwMjE4NGM1YTBiMWUzMjEyMDU1NTBlMDMyNjFiMDk0ZDVjMTcxZjU2MDExOTA0) I first converted from HEX and took the first 5 bytes. I could then 'reverse'-XOR this with the first known 5 bytes of the flag `CHTB{` to get the key `mykey`.

![](/assets/htbca2021/03.png)

1. Then all I had to do was use the key to get the full `plaintext` flag. Here I used [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')XOR(%7B'option':'UTF8','string':'mykey'%7D,'Standard',false)&input=MmUzMTNmMjcwMjE4NGM1YTBiMWUzMjEyMDU1NTBlMDMyNjFiMDk0ZDVjMTcxZjU2MDExOTA0) again.

![](/assets/htbca2021/04.png)

---

## <a name="3"></a>PhaseStream 2
> The aliens have learned of a new concept called "security by obscurity". Fortunately for us they think it is a great idea and not a description of a common mistake. We've intercepted some alien comms and think they are XORing flags with a single-byte key and hiding the result inside 9999 lines of random data, Can you find the flag?

1. Woah, 9999 lines of data ain't going to play nicely with CyberChef. Luckily, after some digging around I found [xortool](https://github.com/hellman/xortool). I grabbed it using `pip3 install xortool`.
1. I knew the aliens used a single-byte key so maybe I could brute force `output.txt`

```
xortool -x -b -l 1 output.txt

-x: input is hex-encoded str
-b: brute force all possible most frequent chars
-l: length of the key
```
1. There's just too much data to manually review so I used `grep` to help me find the flag.

```
grep -ao "CHTB{.*}" xortool_out/*

-a: equivalent to --binary-files=text
-o: show only nonempty parts of lines that match
.*: match everything between {}
```

![](/assets/htbca2021/05.png)

---

## <a name="4"></a>PhaseStream 3
> The aliens have learned the stupidity of their misunderstanding of Kerckhoffs's principle. Now they're going to use a well-known stream cipher (AES in CTR mode) with a strong key. And they'll happily give us poor humans the source because they're so confident it's secure!

Firstly, thanks to [Spymky](https://spymky.dev/){:target="_blank"} who helped me solve this one.

1. We're provided with two ciphertexts

```
464851522838603926f4422a4ca6d81b02f351b454e6f968a324fcc77da30cf979eec57c8675de3bb92f6c21730607066226780a8d4539fcf67f9f5589d150a6c7867140b5a63de2971dc209f480c270882194f288167ed910b64cf627ea6392456fa1b648afd0b239b59652baedc595d4f87634cf7ec4262f8c9581d7f56dc6f836cfe696518ce434ef4616431d4d1b361c

4b6f25623a2d3b3833a8405557e7e83257d360a054c2ea
```

1. And the source code used to generate them

```
from Crypto.Cipher import AES
from Crypto.Util import Counter
import os

KEY = os.urandom(16)


def encrypt(plaintext):
    cipher = AES.new(KEY, AES.MODE_CTR, counter=Counter.new(128))
    ciphertext = cipher.encrypt(plaintext)
    return ciphertext.hex()


test = b"No right of private conversation was enumerated in the Constitution. I don't suppose it occurred to anyone at the time that it could be prevented."
print(encrypt(test))

with open('flag.txt', 'rb') as f:
    flag = f.read().strip()
print(encrypt(flag))
```

1. I can tell that the encryption method is `AES.MODE_CTR`.

> AES-CTR uses the AES block cipher to create a stream cipher. Data is encrypted and decrypted by XORing with the key stream produced by AES encrypting sequential counter block values.

1. By XORing the two ciphertexts together and taking the result, and then XORing that with the plaintext from the source code gives us the flag.

```
cipher(test) = plain(test) XOR [block]
cipher(flag) = plain(flag) XOR [block]
--> [block] = cipher(flag) XOR plain(flag) = cipher(test) XOR plain(test)
==> plain(flag) = cipher(test) XOR cipher(flag) XOR plain(test)
```

![](/assets/htbca2021/06.png)
![](/assets/htbca2021/07.png)

---

## <a name="5"></a>MiniSTRyplace
> Let’s read this website in the language of Alines. Or maybe not?

1. While I waited for the Docker container to start, I downloaded the source code to take a peek under the hood. This web application was using PHP and immediately this block of code stood out to me:

```
<?php
$lang = ['en.php', 'qw.php'];
    include('pages/' . (isset($_GET['lang']) ? str_replace('../', '', $_GET['lang']) : $lang[array_rand($lang)]));
?>
```

1. According to the documentation `str_replace` will replace all occurrences of the search string with the replacement string. In this case any occurrence of `../` would be removed (or replaced with `''`).

1. Any time I see a GET param used to fetch a page (or file) I get excited about a potential `Local File Inclusion`. Knowing the location of the flag meant I just needed a way to bypass the `str_replace` function. I dived into an interactive PHP shell and started trying a few options

![](/assets/htbca2021/08.png)

1. After realising that I just had to add a couple of rogue `../` to get the correct directory traversal the flag was mine.

![](/assets/htbca2021/09.png)