---
layout: post
title:  "Setting Up an SMB and RDP Honeypot"
date:   2019-11-22 7:00:00 +0300
categories: exp honeypot
---

The goal is to setup a Windows SMB and RDP honeypot. However, I did not want to expose the honeypot (Win Srv 2016) to the external/monitoring network which has security onion and internet access. 

I managed to get the following setup running on a small tower server:
![honeypot setup](/assets/honeynetdiagram1.png)

Once an attacker gets into the windows machine, s/he will not have internet access or access to the monitoring network. The attacker is locked in and the network traffic is monitored.

###  Home Router Setup
I setup portforwarding on the home router, such that it forwards all SMB and RDP traffic to the Ubuntu server (192.168.1.4).

### Ubuntu Server Setup
#### RDP Setup
The Ubuntu Server has [pyrdp](https://github.com/GoSecure/pyrdp) running, it MITMs the RDP traffic, ie. intercepts it, collects data, then forwards it the Windows Server (192.168.8.6) which is located at the internal network.

#### SMB Setup
For SMB, I configured iptables to just forward all the SMB traffic to the Windows Server

``` bash
# this resource helped a lot https://askubuntu.com/questions/1050816/ubuntu-18-04-as-a-router
# everything matched what I wanted to achieve expect the iptable rules
# as root
> iptables --flush
> iptables --table nat --flush    
> iptables --delete-chain 
> iptables --table nat --delete-chain 

> vim /etc/default/ufw
DEFAULT_FORWARD_POLICY="ACCEPT"

> vim /etc/ufw/sysctl.conf
net/ipv4/ip_forward=1
net/ipv4/conf/all/forwarding=1

# Again, the modification done to this file is what is different from the resource above
> sudo vim /etc/ufw/before.rules
*nat
:PREROUTING ACCEPT [0:0]
# portforwarding
-A PREROUTING -i ens33 -d 192.168.1.4 -p tcp --dport 445 -j DNAT --to-destination 192.168.8.6:445
# routing
-A POSTROUTING ! -s 127.0.0.1 -j MASQUERADE
COMMIT

> sudo ufw disable && sudo ufw enable
# not the best to allow it for all, but i took the risk
> ufw allow ssh
```

### Windows Server 2016 Setup
On the Windows Server I did the following:
1. Updated it
2. Modified local group policy such that:
   1. Renamed the local administrator username
   2. does not show the username of the last logged in user
   3. does not show information of the logged in user
3. Created user "Administrator" with password "P@ssw0rd"
4. Added user "Administrator" to the "Remote Desktop Users" group
5. Modified Windows Firewall
   1. allowed inbound traffic to port 3389
   2. allowed inbound traffic to port 445
6. Disabled NLA for RDP, pyrdp does not support NLA (at the time of this writing atleast)


### Security Onion Setup
As you can see in the diagram above I've attached two bridged interfaces:
1. Interface 1: sensor, basically this is what will be collecting data and according to security onion's docs, since it is bridged it will monitor everything coming and going from the host machine. This will result in monitoring all traffic that will flow into the network. 
2. Interface 2 (192.168.1.5): management interface. Used to connect to security onion to administer it and do analysis on the data captured/processed from the sensor (alerts, pcaps, logs, the good stuff, etc etc)

### Caveats
1. There is no HIDS on the Windows machine, I am planning on using [Wazuh](https://securityonion.readthedocs.io/en/latest/wazuh.html)
2. Another sensor is needed, it should be placed on the internal network.