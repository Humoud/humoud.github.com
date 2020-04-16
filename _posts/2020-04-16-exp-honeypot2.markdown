---
layout: post
title:  "Setting a Honeypot Network With Monitoring In Mind"
date:   2020-04-16 13:47:00 +0300
categories: exp honeypot monitoring blueteam
---

The goal is to setup a small network of honeypot machines with decent monitoring. The services exposed by the honeypot are legitimate and not tampered with, i.e, there are no honeypot services running. The main focus is the monitoring of Windows machines and seeing the attacks being carried on them via the monitoring services from start to finish.

Services exposed to the internet:
   * FTP (anonymous login enabled)
   * HTTP
   * SMB
   * LDAP
   * RDP

![Network](/assets/honeynetdiagram2.png)

Below is a quick summary of each machine in the network:

* Windows Server 2016 (Domain Controller)
* Windows 10 (Joined to the domain)
* Ubuntu Server 18.04.4 LTS (Web & FTP server)
* PFSense 2.4.5 (Firewall)
* Security Onion 16.04.6.5 (Monitoring)
* Ubuntu Server 18.04.4 LTS (Snapshot Manager)

### Proxy & Logs Network
This network will receive HTTPS traffic from the honeypot network for inspection as well as and logs forwarded by the WinLogBeat agents on the Windows machines in the honeypot network.

### Management Network
SecurityOnion can be accessed from this network. And the machine responsible for reverting the snapshots of the vulnerable machines is on this network.

### Honeypot Network
This one is self explanatory. The vulnerable machines are on this network. Furthermore, a SecurityOnion sniffer (sensor) is attached to this network as well.

### Windows Activity Monitoring
I went with [Sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon) to log activities on the Windows machines. I used SwiftOnSecurity's configuration which can be found on [Github](https://github.com/SwiftOnSecurity/sysmon-config). Many thanks to SwiftOnSecurity.

Then I used [WinLogBeat](https://www.elastic.co/downloads/beats/winlogbeat). I had to make sure that the version of WinLogBeat matches the version of Kibana on running on SecurityOnion. With WinLogBeat you choose what you want to be shipped and where. Sysmon logs are stored in Windows Event Logs under: `Applications and Services > Microsoft > Windows > Sysmon > Operational`. The following is from the WinLogBeat configuration file (winlogbeat.yml):

![winlogbeat.yml](/assets/winlogbeat1.png)
![winlogbeat.yml](/assets/winlogbeat2.png)

I did activate powershell logging and you can see from the configuration file that I am shipping them. But I did not test for it yet properly.

Once Sysmon and WinLogBeat are setup. Then you must configure SecurityOnion to accept Beats traffic. This can be achieved by running `sudo so-allow` then selecting option `b`.

![so_allow](/assets/onion_so_allow1.png)

One last step, new releases of SecurityOnion default to the Minimal installation of Logstash. Meaning you have to add extra configurations which suite your needs manually.

List of configurations I added:
![logstash](/assets/logstash_conf1.png)

How to added them is as follows:

`cd` into `/etc/logstash` first then create symbolic links from the available configurations in the `/etc/logstash/conf.d.available` directory to the `/etc/logstash/conf.d.minimal` directory:

![logstash](/assets/logstash_conf2.png)

After that, restart the logstash service (which is running on a docker container):
`sudo docker stop so-logstash && sudo so-logstash-start`

### Intercepting HTTPS Traffic
With most sites using HTTPS and Firefox pushing for DNS over HTTPS. Decrypting TLS traffic is a must, if you want to get visibility on web activity. For this I went with [PolarProxy](https://www.netresec.com/?page=PolarProxy) and I followed this [tutorial](https://www.netresec.com/?page=Blog&month=2020-01&post=Sniffing-Decrypted-TLS-Traffic-with-Security-Onion) to set it up on SecurityOnion. After setting it up on SecurityOnion and configuring the Windows machines in the honeypot network to trust the CA certificate of PolarProxy, I configured the PFSense firewall to forward all outbound HTTPS traffic from the honeypot network to SecutiyOnion's network interface in the Proxy & Logs Network.

![fw](/assets/fw_portfw_https.png)


### VM Snapshot Management
The Windows machines will be under constant attack, they need to be refreshed frequently. I was not sure how frequently but I went with a one refresh very four hours. I am hosting everything on an ESXi server. I found this [script](https://github.com/vmware/pyvmomi-community-samples/blob/39bd95553df337ae35f1b101c487b37063967555/samples/snapshot_operations.py) which can revert VMs. It uses the `pyvmomi` library by VMWare. Then I setup a cron job on the Ubuntu server in the management network to run the python scripts every four hours.

Here are the cron jobs:

![cronjobs](/assets/cronjobs1.png)

### Canarytokens (Traps)
Each machine (including the Web and FTP server) contains a docx file which sends a beacon when it is opened in Microsoft Word. [Canarytokens](https://canarytokens.org/generate) by thinkst offers this service for free. Many thanks to them.

### Control
We need to make sure that things don't get out of control. To have a level of control on the Windows machines, I did the following:

* Disabled RDP Clip
* Renamed username `Administrator` on the Domain Controller to a nondescriptive name.
* Created a user called `Administrator` with a weak password.
  * Added this user to the Remote Desktop Users group
  * Allowed this user to login to the Domain Controller remotely and locally.
* Created a Domain Admin user with a strong password (Used as backup).
* Prevented Windows machines from showing the username of the last logged in user.

### Issues and Concerns
* The firewall is currently redirecting HTTPS traffic sent on port 443. Thus, if an HTTPS connection is established over another port it will not be forwarded to PolarProxy for decryption.
* I seem to be facing issues with allowing FTP traffic on the firewall. I am using vsftp on the Ubuntu server. I will work on this later.
* I am not sure if Zeek can extract downloaded files from the decrypted HTTPS traffic. I can see the traffic but I am not sure if it can extract files.
* I am yet to understand fully what SwiftOnSecurity's configuration for Sysmon logs, it is currently very helpful but I am not sure if it is missing anything.

### Continuous Monitoring, Fine Tuning, and Upgrades
That is the plan for now. As this network gets exposed to more attacks, I will not harden it. Instead, I will try to improve the logging and detection capabilities.





