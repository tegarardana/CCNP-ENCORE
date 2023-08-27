# Deployment Models

## Introduction to Wireless Networks

Wireless networks…we have to love a world without wires, right? Or maybe not? The usage of wireless networks has exploded in the last couple of years and even replaced desktops with wired connections. Go to any large company or hotel, and they’ll have a guest wireless network you can use to access the internet. In the 1990s, if you wanted to buy a laptop with the same specifications as a desktop you had to pay twice the price. Nowadays I think laptops are cheaper than desktops and with all the smartphones/tablets out there we have become even more mobile!

So what are the differences between wired and wireless networks? Obviously, we don’t use cables but we are using radio waves to transmit data. Remember CSMA/CD, which we used for our half-duplex networks? Wireless is half-duplex since we are sending and receiving on the same frequency. This means we can get collisions if we use wireless networks, but it’s rather hard to detect whether there have been two wireless signals that bumped into each other somewhere in the air. We have a protocol that deals with this called **CSMA/CA, which stands for Carrier Sense Multi Access / Collision Avoidance**.

What other issues do we have to deal with?

* **Coverage**: You’ll need to think about the placement of access points and the frequencies you are going to use to get optimal coverage. Different materials will affect your signal. Implementing a wireless network in a large open room is easier than installing one on a yacht which is one huge metal cage.
* **Interference**: There’s so much going on on the 2.4 and 5 GHz frequencies that you will get interference, weakening your signal quality.
* **Privacy**: Our data “flies around in the air” which means we have no way of securing our physical layer, we need to make sure we have strong authentication and encryption.
* **Regulations**: Each country has regulations you have to deal with. Think about signal strength, allowed frequencies and so on.

Many things can go wrong with your wireless signal:

* **Reflection**
* **Scattering**
* **Absorption**

Reflection is when your wireless signal bounces off the material. Metal is a very good example of this. It’s very hard to get your wireless signal through a metal ceiling or elevator since the signal just bounces off.  Scattering means your wireless signal hits a surface and “breaks” apart in multiple pieces leaving the original signal far weaker. Absorption happens when a material absorbs our wireless signal. Examples of absorption are water and the human body…absorption is terrible for your wireless signal since there’s not much left after passing through this material!

Higher frequencies will give you higher data rates, the higher your frequency, the more “waves” you have in a given time cycle:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/wireless-waves.png" alt=""><figcaption></figcaption></figure>

If you want a more visual representation of wireless signals and how they behave, there’s a nice website that demonstrates this. It’s called [EMANIM, ](https://emanim.szialab.org/index.html)and you can find it here:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/emanim-example.png" alt=""><figcaption></figcaption></figure>

You can see in real-time what the difference is between low and high frequencies, absorption, and so on.

If you have to deal with wireless, there are some organizations you have to deal with (more or less):

* ITU-R: International Telecommunication Union-Radiocommunication Sector, these guys regulate the radio frequency we use for wireless worldwide.
* IEEE: Institute of Electrical and Electronic Engineers: These are the people that create all the standards we use today. Wireless is documented in the 802.11 standards.
* Wi-Fi Alliance: The Wi-Fi Alliance is a non-profit organization that promotes wireless usage. Have you ever seen one of those WiFi-certified stickers on your new wireless router? That’s the Wi-Fi Alliance.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/wifi-certified.jpg" alt=""><figcaption></figcaption></figure>

You are not allowed to transmit on any given frequency you like. The International Telecommunication Union-Radiocommunication Sector (ITU-R) has determined a couple of frequencies we can use for our wireless networks.

These frequency bands are called the ISM band which stands for Industrial, Scientific, and Medical. Everyone can use these frequencies without the requirement of getting a license. That’s also the downside since everyone is using them, you are likely to get interference.

The ISM band has the following frequencies:

* 902-928MHz: We don’t use this low frequency for our Wi-Fi equipment.
* 2.4-2.4835 GHz: This is a frequency we use for 802.11b, 802.11g and 802.11n
* 5 GHz: There are a couple of frequencies on the 5 GHz band we can use. 802.11a and 802.11n operate here.

What standards do we have? There’s 802.11a, 802.11b, 802.11g, and 802.11n. What are the differences? First, let me show you this table:

<table><thead><tr><th width="108"></th><th width="104">802.11a</th><th width="136">802.11b</th><th width="113">802.11g</th><th width="151">802.11n</th></tr></thead><tbody><tr><td>Frequency</td><td>5GHz</td><td>2.4GHz</td><td>2.4GHz</td><td>2.4 and 5GHz</td></tr><tr><td>Channels</td><td>23</td><td>3</td><td>3</td><td>depends</td></tr><tr><td>Data Rate</td><td>Up to 54Mbit</td><td>Up to 11Mbit</td><td>Up to 54Mbit</td><td>Up to 300-600Mbit</td></tr></tbody></table>

What do we have here? You can see 802.11b is the slowest standard we have, which can only get to 11Mbit. 802.11a is faster but operates on the 5GHz band. We get speeds up to 54Mbit here. The same technique used for 802.11a is being used for 802.11g but is now on the 2.4GHz band. This makes 802.11b and 802.11g compatible since they operate on the same frequency.

802.11n is another story…a lot of things have been changed to increase its performance. You can get from 300 or 600Mbit, and it can operate on the 2.4 and 5GHz frequency bands.

These data rates are at the physical layer, by the way…on a real network, you will never get a performance like that which makes it a bit misleading. For example, 802.11a is 54Mbit at the physical layer. Think about the OSI model again for a minute. Let’s say we have a computer that wants to transmit something to another computer using wireless:

1. We have an application that wants to send data so it will create a header and put it in front of the data.
2. It goes through the presentation and session layer, I don’t care about those for the moment.
3. At the transport layer, we’ll use TCP in our example, which will add another header.
4. We are using IP, so we need to create an IP packet that adds another header.
5. Our IP packet will be put in a wireless Ethernet frame (wireless Ethernet frames are different from wired Ethernet frames).
6. Everything will be launched into the air as radio waves.

Depending on the size of your data (we call this payload), you can have up to 50% of headers (TCP, IP, FRAME) compared to the actual payload. Half of your bandwidth is wasted because of “headers.” This is something that has been changed for 802.11n.

For 802.11b and 802.11g, you can see there are only three channels. If you ever configured your home wireless router, you might have noticed you could configure 11 channels. In reality, there are only three channels that are non-overlapping.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ISM-channels-24ghz.png" alt=""><figcaption></figcaption></figure>

You can use channels 1,6 and 11 without having interference. You can use other channels, but you need to make sure there are 5 channels in between, or you’ll have interference. This is one of the reasons we can also use the 5GHz range…we have SO much more space there. The trade-off is that our coverage on the 2.4GHz band is far better than on 5GHz.

802.11n can operate on both 2.4GHz and 5GHz, and the number of channels “depends.” This is because 802.11n can do something cool called “channel-bonding”. Basically, you use twice the bandwidth to get twice the performance…very cool but this leaves little space left for other wireless devices. If you use channel bonding on the 2.4GHz frequency, you use so much space that there’s no non-overlapping channel left!

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/80211n-on-ISM-24ghz.png" alt=""><figcaption></figcaption></figure>

Last but not least. The most important (or perhaps I should say the most concerning) topic about wireless…security.

Security is hot, and there are all kinds of wild stories/myths going around about whether wireless is secure or not. You can make wireless very secure if you want, and we’ll discuss the different security protocols and what is working or not.

Years ago, when wireless was getting popular we had three methods that were supposed to increase our wireless networks:

* Filtering MAC addresses.
* Hiding your SSID (name of the network).
* Enabling WEP encryption.

These three methods are all worthless, and I will show you why. First of all, filtering MAC addresses doesn’t add anything to security since MAC addresses are always sent in clear text. That’s right! They are not encrypted even if you are using WPA or WPA2. The following picture is a wireless frame with WPA2 encryption:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/wireshark-80211-wpa2-header.png" alt=""><figcaption></figcaption></figure>

As you can see, our destination and source MAC addresses are in clear text. In case you are wondering where I see this is a WPA2 frame…at the bottom, you see “CCMP parameters.” **CCMP stands for Cipher Chaining Messaging Protocol** which is what WPA2 uses for encryption.

Hiding the SSID (name of your network) is also nonsense since there are three frame types where you will find the SSID:

* Beacon: Your wireless access point will send beacons, broadcasting its SSID and other information like data rates.
* Probe request and probe response: These are used when a client wants to connect to your wireless access point.

If you hide your SSID, it’s only disabled in the beacon. It’s still in the probe request and probe response in clear-text ready to be sniffed by a wireless hacker.

Talking about sniffers…running a sniffer/packet analyzer like Wireshark on a company network is NOT a good idea unless you are given clearance to do so. It’s possible to detect sniffers like Wireshark on your network, and the security team of your company is not going to be happy with you. You can detect Wireshark usage on a wired network.

Running a wireless sniffer, however, is a different story. You are just capturing whatever is flying around in the air, and there’s no way to detect that someone is doing this!

WEP encryption is unsafe. I won’t go into the details of it, but WEP uses a weak encryption algorithm (RC4) and uses static keys as part of the encryption process. WEP networks can be hacked in 5-10 minutes, no matter if you use a 64,128 or 256-bit key. So much for **Wired Equivalent Privacy (the acronym WEP stands for).**

The next protocol in line is WPA (version 1). WPA uses the same encryption (RC4) as WEP, but a lot has changed to increase security. WEP uses static keys whereas WPA uses TKIP (Temporal Key Integrity Protocol) as the input for RC4. Simply said…dynamic keys instead of static keys that WEP has. Despite what you might read on the Internet, WPA is still secure, and nobody has “fully hacked” WPA, unlike WEP, which is completely wide open on the table.

Since WEP and WPA both use RC4 encryption, all old hardware that only supported WEP can be upgraded to use WPA but not WPA 2. WPA 2 is completely redesigned. Instead of using the RC4 encryption algorithm, it works by using AES. AES is the most secure encryption algorithm up-to-date.

So are WPA and WPA 2 safe from hackers? Yes and no. Nobody so far has managed to find a weakness in the protocols, so it’s safe, but it depends on the key you are using. There are two methods of using a key for WPA and WPA 2.

* Preshared key: This is what you probably use at home. You made up a key that’s being used for your wireless encryption.
* 802.1x and EAP: This is what you use for serious wifi setups since you can authenticate users.

Using a Preshared key is easy, but you don’t have any control. You don’t know who has your key, and it’s easy to share it. It’s also being saved in clear text in your Windows registry. If you have a strong Preshared key, it’s impossible to break it. Short Preshared keys or using words you might find in the dictionary are easy to crack.

The options you have to hack WPA or WPA 2 are to use a dictionary file (which is a big list with all words from the dictionary) or brute force (which tries every possible combination one by one). If you use a Preshared key with enough characters and enough complexity, you should be reasonably safe. In 2017, a serious weakness was discovered in WPA2 using key reinstallation attacks (KRACKs). Operating systems have been patching this vulnerability but WPA2 is showing its age.

The most secure method is 802.1X, also known as port-based control. This is something you can do for wireless but also wired networks. The idea behind it is that users need to authenticate themselves before they get access to the network. You don’t even receive an IP address from the DHCP server…the only thing you are allowed to do is send authentication information.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/wireless-8021x-radius.png" alt=""><figcaption></figcaption></figure>

On the left side, we have the supplicant, which is our computer or laptop, who has to authenticate herself using authentication information. You can use different types like username/password, certificates, or perhaps a token or OTP (one-time password). The type of authentication you use depends on the EAP type. There’s an EAP type for username/password, something for certificates on the client and/or server side, and so on. In the middle, we see our wireless access point, which is the authenticator. If this were a wired network, this would be our switch. The authenticator is responsible for getting the authentication information of the user and passing them along to the authentication server, which is normally a RADIUS server.

There are two major advantages to using this setup for wireless networks:

* You authenticate per user so you know who uses the wireless network compared to a preshared key that “anyone could have.”
* The authentication server will generate the “Primary master key” that WPA or WPA 2 uses. Simply said…different encryption per user.

If you somehow managed to brute force WPA or WPA 2 encryption for a single user, then all other users are safe because they use different primary master keys from the RADIUS server. If you use the preshared key, the preshared key becomes the primary master key for WPA or WPA 2.

I hope you enjoyed it. If you have any more questions, please leave a comment!
