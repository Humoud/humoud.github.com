layout: post
title:  "Analysis and Detection: Exfil Via TLS Cipher Suites"
date:   2021-12-04 00:00:00 +0300
categories: dfir
---


About a month ago, Mohammed Aldoub AKA [Voulnet](https://twitter.com/Voulnet/status/1454129822620393472?s=20) shared a very interesting idea about exfiltrating data via TLS cipher suites (many thanks Mohammed!) on [Twitter](https://twitter.com/Voulnet/status/1454129822620393472?s=20). This had me thinking about detection and JA3 hashes came to mind with hopes of it shedding light on abnormal behaviour on endpoints.

I used the following to simulate the attack:
1. I used the following repo to pull off the exfil technique (thanks adeemm!): [github repo](https://github.com/adeemm/ex-509).
2. Spun up a digital ocean instance to simulate an attacker's infrastructure.
3. Ran a Windows 10 VM to simulate the victim.
4. Setup and ran a [Security Onion](https://securityonionsolutions.com/) VM to do analysis and detection on the network traffic.

# Simulation
## Victim Machine
Once the setup was ready. I started the simulation by running the exfiltration tool on the victim. I pointed the tool towards:
1. The file I wanted to exfil (secrets.txt)
2. Digital ocean instance (the IP address in the image)
3. Port 443, because the server is listening on that port (more on that shortly)

![victim](/assets/tls_exfil_1/client_file_exfil.PNG)

## Attacking Server
On our attacking server, we are already running the server part of the tool and setting it to listen on port 443.

We see the tool receiving connections:

![server](/assets/tls_exfil_1/server_cmd.PNG)

And we can see the exfiltrated files arrived in .bin files:

![server](/assets/tls_exfil_1/server_file_exfil.PNG)

# Analysis
Using security onion, I extracted the packets which contain the TLS client hello packet.

Security Onion:

![SO](/assets/tls_exfil_1/so_exfil.PNG)

Wireshark:

![server](/assets/tls_exfil_1/wireshark_exfil.PNG)

The source is the victim machine and the destination is our attacking server, note the cipher suites they are holding the data we are exfiltrating.

Comparing a legitimate TLS client hello (left) to one which is exfiltrating data (right) we see some fields missing. The evil client hellos are missing data:

![missing fields](/assets/tls_exfil_1/tls_missing.PNG)

I will not focus on client hellos missing data in this specific analysis because I wish to find a detection method which covers all exfiltration tools which leverage the technique in question. I am unsure if it is possible, but I fear that the script can be modified to include the missing fields or mimic them.

# Detection Attempt 1: NIDS
By default, the Suricata running on Security Onion had no rules for this use case. I am no NIDS ninja but I tried to make a rule and failed miserably. That is because I was not able to target the cipher suites in the client hello packets. I am not sure if it is possible...

I did make a super simple rule to make things easier when I moved forward with my analysis and that was to make a rule to detect all TLS client hellos, very noisy and impractical I know, I wanted to at least see **all** TLS client hellos. I will add the rule down below in hopes that it might help someone one day:

```
alert tls $HOME_NET any -> $EXTERNAL_NET any (msg:"TLS CLIENT HELLO Detected"; ssl_state:client_hello; sid:9000001; rev:2; metadata:deployment Perimeter, former_category EXFILTRATION, signature_severity Major, updated_at 2021_12_04, created_at 2021_12_04;)
```

Resource: [Adding suricata rules to security onion](https://docs.securityonion.net/en/2.3/local-rules.html)


# Detection Attempt 2: JA3 Hashes
The use case is as follows:
- One victim IP
- One attacker IP
- Multiple JA3 hashes

And I created a visualization on Security Onion's Kibana which shows unique JA3 hashes counts for each destination IP address. This visualization becomes useful after filtering for a specific source IP address which will be the victim.

![server](/assets/tls_exfil_1/kibana_ja3.PNG)


Success? In this simulated controlled environment? yes.

# Conclusion
1. If we see multiple packets originating from an endpoint, each with a unique JA3 hash and the destination is the same, then we should alert on it.
2. More research is needed

# Thoughts
1. Is it possible to detect malformed cipher suites?
   - This will make detection easier

# Concerns
1. This technique can be made even stealthier by incorporating other techniques to stay under the radar.