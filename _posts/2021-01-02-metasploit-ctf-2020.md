---
layout: post
title: "Metasploit CTF 2020"
date: 2021-01-02
---

[https://metasploitctf.com/](https://metasploitctf.com/)

This was my first CTF challenge. I couldn't dedicate too much time over the specific weekend, but I was able to contribute in a very small way to two of the challenges. A big shout out to the rest of my team who dominated the challenges leaving us ranked 39th out of 413 teams.

### Setup
The target was not directly accessible, so all teams had to use a supplied Kali jump box.

> A jump box functions like a proxy server and is a way to isolate access to a private network. It is usually a computer that is connected to two networks and has two network cards. ... The jump box is then configured to correctly route traffic between the two networks.

This required a bit of configuration on my local machine:
1. Download the supplied SSH key
2. `chmod 600 metasploit_ctf_kali_ssh_key.pem`
3. `echo "socks4  127.0.0.1 1088" >> /etc/proxychains.conf`
4. `ssh -D 1088 -i metasploit_ctf_kali_ssh_key.pem kali@xxx.xxx.xxx.xxx`

With the SSH tunnel now configured. I could access web content by proxying through 127.0.0.1:1088 or by prefixing any terminal commands with `proxychains`.  

Example: `proxychains nmap -sC -sV xxx.xxx.xxx.xxx`   

### Challenge 1
This involved brute forcing a web login form, but with a slight twist. The challenge hinted towards a valid username and it became apparent that a valid username resulted in a delayed POST response. I decided to write a simple Python script that would iterate over a wordlist of common usernames and record the response times. Any username that resulted in longer than usual response time was likely to be valid.  

Script: [response-timer.py](https://github.com/parity0x1/ctf/blob/main/response-timer.py)  


### Challenge 2
A simple (but impossible for humans) game played via `nc` where commands were sent to move the player (^) left or right in an attempt to avoid colliding with approaching 0's. The longer you survived the faster the, we'll call them asteroids, approached. Despite banging away at automating with Python for about 2 hours, I just wasn't able to finish this challenge in time.

Script: [auto-play.py](https://github.com/parity0x1/ctf/blob/main/auto-play.py)  

UPDATE: Seems like I was on the right track, which is quite encouraging. [See the solution here](https://gist.github.com/busterb/2fcd6f95acc89c0b85ef2d08b89930ae)
