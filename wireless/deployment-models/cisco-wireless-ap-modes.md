# Cisco Wireless AP Modes

Many Cisco APs can operate in autonomous or lightweight mode; this depends on the image that you run.

An AP that serves wireless clients is in local mode. Besides local mode, there are other AP modes. In this lesson, we’ll take a look at each AP mode.

## AP Modes

### Local

Local mode is the default mode; it offers a BSS on a specific channel. When the AP doesn’t transmit wireless client frame, it’s still doing something behind the scenes. The AP scans other channels to:

* Measure noise
* Measure interference
* Discover rogue devices
* Check for matches against IDS events

### Monitor

An AP in monitor mode **doesn’t transmit** at all. It’s a dedicated sensor that:

* Checks Intrusion Detection System (IDS) events
* Detects rogue APs
* Determines the position of wireless stations

Because the AP is only in monitor mode, it won’t broadcast an SSID so clients are unable to connect to the AP.

### FlexConnect

It’s possible to connect a local mode AP at a remote branch to the HQ’s WLC. This works, but it’s not a good idea. First of all, the AP **encapsulates all wireless client data through the CAPWAP tunnel** over the WAN link. Secondly, when the WAN link is down, your wireless network at the branch site is offline too.

FlexConnect is an AP mode for situations like the one above. The AP can **locally switch traffic between a VLAN and SSID** when the CAPWAP tunnel to the WLC is down.

### Sniffer

An AP in sniffer mode dedicates its time to receive 802.11 wireless frames. The AP becomes a **remote wireless sniffer; you** can connect to it from your PC with an application like Wildpackets Omnipeek or Wireshark. This can be useful if you want to troubleshoot a problem and you can’t be on-site. When an AP is in sniffer mode, it won’t broadcast an SSID so clients can’t connect to the AP.

### Rogue Detector

Rogue detector mode makes the AP detect rogue devices full-time. The AP checks for MAC addresses it sees **in the air and on the wired network**. When the AP is in rogue detector mode, it can switch between rogue detection and serving clients. The AP can still broadcast an SSID and clients can connect to the AP.

### Bridge/Mesh

The AP becomes a dedicated point-to-point or point-to-multipoint bridge. Two APs in bridge mode can connect two remote sites. Multiple APs can also form an indoor or outdoor mesh. You can’t connect to the bridge with clients.

### Flex plus Bridge

The AP can operate in either FlexConnect or Bridge/Mesh mode. This AP mode **combines the two; it** allows APs in mesh mode to use FlexConnect capabilities.

### SE-Connect

An AP in SE-Connect mode dedicates its radios to **spectrum analysis on all wifi channels**. You can connect to the AP from your PC with applications like MetaGeek Chanalyzer or Cisco Spectrum Expert. This is useful if you want to remotely discover interference sources that you can’t solve with a sniffer. The AP won’t broadcast an SSID so clients can’t connect to it.
