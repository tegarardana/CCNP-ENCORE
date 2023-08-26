# OSPFv3 Authentication and Encryption

OSPFv3 doesn’t have an authentication field in its header like OSPFv2 does, instead it relies on IPsec to get the job done.

IPsec supports two encapsulation types. The first one is AH (Authentication Header) which as the name implies, authenticates the header. The other encapsulation type is ESP (Encapsulating Security Payload) which encrypts packets. We can use both for OSPFv3 so besides authentication, encryption is also a possibility.

In this lesson I’ll show you how to configure both options.

## Configuration

We will use the following topology for this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/ospfv3-r1-r2-area-0.png" alt=""><figcaption></figcaption></figure>

We only need two routers for this demonstration. I will only use the link-local IPv6 addresses on these two routers. Let’s enable OSPFv3:

<pre><code>R1 &#x26; R2#
<strong>(config)#interface FastEthernet 0/0
</strong><strong>(config-if)#ipv6 ospf 1 area 0
</strong></code></pre>

Now we can play with authentication…

### IPsec Authentication

To get started we have to use the **ipv6 ospf authentication** command:

<pre><code><strong>R1(config)#interface FastEthernet 0/0
</strong><strong>R1(config-if)#ipv6 ospf authentication ?
</strong>ipsec Use IPsec authentication
null Use no authentication
</code></pre>

Since we want authentication, we’ll select ipsec:

<pre><code><strong>R1(config-if)#ipv6 ospf authentication ipsec ?
</strong>  spi  Set the SPI (Security Parameters Index)
</code></pre>

First we have to choose a SPI. You can pick any number you like but it has to match on both routers. Let’s pick the lowest available number (256):

<pre><code><strong>R1(config-if)#ipv6 ospf authentication ipsec spi 256 ?
</strong>  md5   Use MD5 authentication
  sha1  Use SHA-1 authentication
</code></pre>

Now we can choose what authentication we would like, MD5 or SHA1. SHA1 is more secure so let’s select that:

<pre><code><strong>R1(config-if)#ipv6 ospf authentication ipsec spi 256 sha1 ?
</strong>  0           The key is not encrypted (plain text)
  7           The key is encrypted
  Hex-string  SHA-1 key (40 chars)
</code></pre>

Now we have to type in a key string ourselves. Normally IPsec uses IKE (Internet Key Exchange) for the security association between two devices. However since we can have multiple OSPFv3 neighbors on a single segment we can’t use IKE and we’ll have to use a static key instead.

For this example I will use an [online SHA1 generator](http://www.sha1-online.com/) to generate a key but for a production network you really should use a safer method to generate a key. Let’s enter that key:

<pre><code><strong>R1(config)#interface FastEthernet 0/0
</strong><strong>R1(config-if)#ipv6 ospf authentication ipsec spi 256 sha1 A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
</strong></code></pre>

As soon as you do this the OSPFv3 neighbor adjacency will drop so let’s copy and paste the same line on R2:

<pre><code><strong>R2(config)#interface FastEthernet 0/0
</strong><strong>R2(config-if)#ipv6 ospf authentication ipsec spi 256 sha1 A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
</strong></code></pre>

That should do the job.

{% hint style="info" %}
It’s also possible to configure authentication for the entire area. If you want this you’ll have to use the **`area 0 authentication`** command under the OSPFv3 process.
{% endhint %}

let’s verify our work:

<pre><code><strong>R1#show ipv6 ospf interface FastEthernet 0/0 | include auth
</strong>  SHA-1 authentication SPI 256, secure socket UP (errors: 0)
</code></pre>

<pre><code><strong>R2#show ipv6 ospf interface FastEthernet 0/0 | include auth
</strong>  SHA-1 authentication SPI 256, secure socket UP (errors: 0)
</code></pre>

If you look at the OSPF specific information on the interface then you can see that authentication has been enabled. Since we are using IPsec, you can also check the security associations:

<pre><code><strong>R1#show crypto ipsec sa
</strong>
interface: FastEthernet0/0
    Crypto map tag: (none), local addr ::

   IPsecv6 policy name: OSPFv3-1-256 
   IPsecv6-created ACL name: FastEthernet0/0-ipsecv6-ACL

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (FE80::/10/89/0)
   remote ident (addr/mask/prot/port): (::/0/89/0)
   current_peer :: port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 50, #pkts encrypt: 50, #pkts digest: 50
    #pkts decaps: 31, #pkts decrypt: 31, #pkts verify: 31
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: ::,
     remote crypto endpt.: ::
     path mtu 1500, ipv6 mtu 1500, ipv6 mtu idb FastEthernet0/0
     current outbound spi: 0x100(256)
     PFS (Y/N): N, DH group: none

     inbound esp sas:

     inbound ah sas:
      spi: 0x100(256)
        transform: ah-sha-hmac ,
        in use settings ={Transport, }
        conn id: 2001, flow_id: NETGX:1, sibling_flags 80000001, crypto map: (none)
        no sa timing
        replay detection support: N
        Status: ACTIVE

     inbound pcp sas:

     outbound esp sas:

     outbound ah sas:
      spi: 0x100(256)
        transform: ah-sha-hmac ,
        in use settings ={Transport, }
        conn id: 2002, flow_id: NETGX:2, sibling_flags 80000001, crypto map: (none)
        no sa timing
        replay detection support: N
        Status: ACTIVE

     outbound pcp sas:
</code></pre>

Above you can see our SPI number and that we are using SHA authentication. There’s one more useful command:

<pre><code><strong>R1#show crypto ipsec policy 
</strong>Crypto IPsec client security policy data

Policy name:      OSPFv3-1-256
Policy refcount:  1
Inbound  AH SPI:  256 (0x100)
Outbound AH SPI:  256 (0x100)
Inbound  AH Key:  A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
Outbound AH Key:  A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
Transform set:    ah-sha-hmac
</code></pre>

This gives us a nice overview with our authentication method, SPI and keys. If you are interested, here’s a wireshark capture of our authenticated OSPFv3 packets:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/09/wireshark-ipv6-ospfv3-ah.png" alt=""><figcaption></figcaption></figure>

Above you can see the authentication header. If you want to take a look for yourself then you can find the [capture file here](https://cdn.networklessons.com/wp-content/uploads/2015/07/wireshark-ospfv3-ah.pcap).

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ipv6 unicast-routing 
!
interface fastEthernet 0/0
 ipv6 enable
 ipv6 ospf 1 area 0
 ipv6 ospf authentication ipsec spi 256 sha1 A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
!
ipv6 router ospf 1
 router-id 1.1.1.1
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ipv6 unicast-routing 
!
interface fastEthernet 0/0
 ipv6 enable
 ipv6 ospf 1 area 0
 ipv6 ospf authentication ipsec spi 256 sha1 A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
!
ipv6 router ospf 1
 router-id 2.2.2.2
!
end
```
{% endtab %}
{% endtabs %}

### IPsec Encryption

Let’s take a look at the second method, using IPsec ESP to authenticate _and_ encrypt OSPFv3 traffic. Let’s get rid of the current IPsec AH configuration:

<pre><code>R1 &#x26; R2#
<strong>(config)#interface FastEthernet 0/0
</strong><strong>(config-if)no ipv6 ospf authentication ipsec spi 256 sha1 A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
</strong></code></pre>

Now we can enable ESP, there’s a different command we have to use:

<pre><code><strong>R1(config-if)#ipv6 ospf encryption ipsec spi 256 esp ?
</strong>  3des     Use 3DES encryption
  aes-cbc  Use AES-CBC encryption
  des      Use DES encryption
  null     ESP with no encryption
</code></pre>

This time you have to use the ipv6 ospf encryption command. You still have to select the SPI number and above you can see the options for ESP. Let’s select AES since it’s the most secure encryption method:

<pre><code><strong>R1(config-if)#ipv6 ospf encryption ipsec spi 256 esp aes-cbc ?
</strong>  128  Use 128 bit key
  192  Use 192 bit key
  256  Use 256 bit key
</code></pre>

We get to select the size of our key. Let’s go for 256-bit:

<pre><code><strong>R1(config-if)#ipv6 ospf encryption ipsec spi 256 esp aes-cbc 256 ?
</strong>  0           The key is not encrypted (plain text)
  7           The key is encrypted
  Hex-string  256bit key (64 chars)
</code></pre>

Now we have to enter the 256-bit key ourselves. For this example I used [this website](https://www.grc.com/passwords.htm) to generate one but for a production network it’s probably best to use something a bit more secure. Let’s enter our key:

<pre><code><strong>R1(config-if)#ipv6 ospf encryption ipsec spi 256 esp aes-cbc 256 FDCEA619E8AA73F517F6EA997D7D782F3EB47B4FA425AA0AD19D73C2A7FBD85B sha1 A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
</strong></code></pre>

Once you enter the encryption key you also have to specify the authentication algorithm and key. Just like the previous example I used SHA1 for this. Let’s copy and paste this entire line to R2 as well:

<pre><code><strong>R2(config)#interface FastEthernet 0/0
</strong><strong>R2(config-if)#ipv6 ospf encryption ipsec spi 256 esp aes-cbc 256 FDCEA619E8AA73F517F6EA997D7D782F3EB47B4FA425AA0AD19D73C2A7FBD85B sha1 A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
</strong></code></pre>

Let’s verify our work:

<pre><code><strong>R1#show ipv6 ospf interface FastEthernet 0/0 | include auth
</strong>  AES-CBC-256 encryption SHA-1 auth SPI 256, secure socket UP (errors: 0)
</code></pre>

Here we can see that we are using AES for encryption ahd SHA1 for authentication. Let’s check our IPsec SA:

<pre><code><strong>R1#show crypto ipsec sa
</strong>
interface: FastEthernet0/0
    Crypto map tag: (none), local addr FE80::21D:A1FF:FE8B:36D0

   IPsecv6 policy name: OSPFv3-1-256 
   IPsecv6-created ACL name: FastEthernet0/0-ipsecv6-ACL

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (FE80::/10/89/0)
   remote ident (addr/mask/prot/port): (::/0/89/0)
   current_peer :: port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 80, #pkts encrypt: 80, #pkts digest: 80
    #pkts decaps: 41, #pkts decrypt: 41, #pkts verify: 41
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: FE80::21D:A1FF:FE8B:36D0,
     remote crypto endpt.: ::
     path mtu 1500, ipv6 mtu 1500, ipv6 mtu idb FastEthernet0/0
     current outbound spi: 0x100(256)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x100(256)
        transform: esp-256-aes esp-sha-hmac ,
        in use settings ={Transport, }
        conn id: 2003, flow_id: NETGX:3, sibling_flags 80000001, crypto map: (none)
        no sa timing
        IV size: 0 bytes
        replay detection support: N
        Status: ACTIVE

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x100(256)
        transform: esp-256-aes esp-sha-hmac ,
        in use settings ={Transport, }
        conn id: 2004, flow_id: NETGX:4, sibling_flags 80000001, crypto map: (none)
        no sa timing
        IV size: 0 bytes
        replay detection support: N
        Status: ACTIVE

     outbound ah sas:

     outbound pcp sas:
</code></pre>

Above you can see that some packets have been encrypted / decrypted and that we are using IPsec ESP / AH. If you only want a quick overview, take a look below:

<pre><code><strong>R1#show crypto ipsec policy 
</strong>Crypto IPsec client security policy data

Policy name:      OSPFv3-1-256
Policy refcount:  1
Inbound  ESP SPI:        256 (0x100)
Outbound ESP SPI:        256 (0x100)
Inbound  ESP Auth Key:   A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
Outbound ESP Auth Key:   A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
Inbound  ESP Cipher Key: FDCEA619E8AA73F517F6EA997D7D782F3EB47B4FA425AA0AD19D73C2A7FBD85B
Outbound ESP Cipher Key: FDCEA619E8AA73F517F6EA997D7D782F3EB47B4FA425AA0AD19D73C2A7FBD85B
Transform set:    esp-256-aes esp-sha-hmac
</code></pre>

Let me also show you a Wireshark capture of some encrypted OSPFv3 traffic:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/packet-capture-ospfv3-esp.jpg" alt=""><figcaption></figcaption></figure>

We can recognize it as OSPFv3 traffic because we see the destination is FF02::5 but that’s it. Everything else is encrypted.

If you want to take a look for yourself, you can find the capture file below:

[OSPFv3 ESP](https://www.cloudshark.org/captures/e8cf17c06f7c)

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ipv6 unicast-routing 
!
interface fastEthernet 0/0
 ipv6 enable
 ipv6 ospf 1 area 0
 ipv6 ospf encryption ipsec spi 256 esp aes-cbc 256 FDCEA619E8AA73F517F6EA997D7D782F3EB47B4FA425AA0AD19D73C2A7FBD85B sha1 A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
!
ipv6 router ospf 1
 router-id 1.1.1.1
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ipv6 unicast-routing 
!
interface fastEthernet 0/0
 ipv6 enable
 ipv6 ospf 1 area 0
 ipv6 ospf encryption ipsec spi 256 esp aes-cbc 256 FDCEA619E8AA73F517F6EA997D7D782F3EB47B4FA425AA0AD19D73C2A7FBD85B sha1 A5DEC4DD155A695A8B983AACEAA5A97C6AECB6D1
!
ipv6 router ospf 1
 router-id 2.2.2.2
!
end
```
{% endtab %}
{% endtabs %}

Hopefully this lesson has been useful to you to understand OSPFv3 authentication and encryption. If you have any questions, feel free to leave a comment!
