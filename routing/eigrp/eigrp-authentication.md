# EIGRP Authentication

Routing protocols can be configured to prevent receiving false routing updates, and EIGRP is no exception. If you don’t use authentication and you are running EIGRP, someone could try to form an EIGRP neighbor adjacency with one of your routers and try to mess with your network…we don’t want that to happen right?

EIGRP supports **MD5 authentication and** [**SHA authentication**](https://networklessons.com/eigrp/eigrp-sha-authentication)**.** There is no plaintext authentication.

What does authentication offer us?

* Your router will authenticate the source of each routing update packet it will receive.
* Prevents false routing updates from sources that are not approved.
* Ignore malicious routing updates.

A potential hacker could be sitting on your network with a laptop, booting up a virtual Cisco router and trying the following things:

* Establish a neighbor adjacency with one of your routers and advertise junk routes.
* Send malicious packets and see if you can drop the neighbor adjacency of one of your authorized routers.

## Configuration

To configure EIGRP authentication, we need to do the following:

* Configure a key-chain
  * Configure a key ID under the key-chain.
    * Specify a password for the key ID.
    * Optional: specify accept and expire lifetime for the key.

Let’s use two routers and see if we can configure EIGRP MD5 authentication:

\
The configuration for both routers is very basic:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-with-keys-1.png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>R1(config)#interface fastEthernet 0/0
</strong><strong>R1(config-if)#ip address 192.168.12.1 255.255.255.0
</strong>
<strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#network 192.168.12.0
</strong></code></pre>

<pre><code><strong>R2(config)#interface fastEthernet 0/0
</strong><strong>R2(config-if)#ip address 192.168.12.2 255.255.255.0
</strong>
<strong>R2(config)#router eigrp 12
</strong><strong>R2(config-router)#network 192.168.12.0
</strong></code></pre>

The first thing we need to configure is a key-chain:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-keychain.png" alt=""><figcaption></figcaption></figure>

I called mine “MY\_KEY\_CHAIN,” but it can be different on both routers. It doesn’t matter. The Key ID is a value that has to match on both routers, and the key-string is the password which has to match, of course. This is how to configure a key-chain:

<pre><code><strong>R1(config)#key chain MY_KEY_CHAIN
</strong><strong>R1(config-keychain)#key 1
</strong><strong>R1(config-keychain-key)#key-string MY_KEY_STRING
</strong></code></pre>

<pre><code><strong>R1(config)#interface fastEthernet 0/0
</strong><strong>R1(config-if)#ip authentication mode eigrp 12 md5 
</strong><strong>R1(config-if)#ip authentication key-chain eigrp 12 MY_KEY_CHAIN
</strong></code></pre>

First, you must create the key-chain, and then activate it on the interface. The “12” is the AS number of EIGRP. We can enable a debug to see whether authentication is working or not:

<pre><code><strong>R2#debug eigrp packets 
</strong>EIGRP Packets debugging is on
    (UPDATE, REQUEST, QUERY, REPLY, HELLO, IPXSAP, PROBE, ACK, STUB, SIAQUERY, SIAREPLY)

R2# EIGRP: FastEthernet0/0: ignored packet from 192.168.12.1, opcode = 5 (authentication off or key-chain missing)
</code></pre>

You can check if your configuration is correct by using `debug eigrp packets`. You can see that we received a packet with MD5 authentication, but I didn’t enable MD5 authentication yet on R2.

Let’s fix that:

<pre><code><strong>R2(config)#key chain MY_KEY_CHAIN
</strong><strong>R2(config-keychain)#key 1
</strong><strong>R2(config-keychain-key)#key-string MY_KEY_STRING
</strong>
<strong>R2(config)#interface fastEthernet 0/0
</strong><strong>R2(config-if)#ip authentication mode eigrp 12 md5
</strong><strong>R2(config-if)#ip authentication key-chain eigrp 12 MY_KEY_CHAIN
</strong></code></pre>

Right away, I can see that the EIGRP neighbor adjacency is working:

```
R2# %DUAL-5-NBRCHANGE: IP-EIGRP(0) 12: Neighbor 192.168.12.1 (FastEthernet0/0) is up: new adjacency
```

What if I entered a wrong key-string?

<pre><code><strong>R1(config)#key chain MY_KEY_CHAIN
</strong><strong>R1(config-keychain)#key 1
</strong><strong>R1(config-keychain-key)#key-string WRONG_KEY_STRING
</strong></code></pre>

You’ll see an error like this in your debug messages:

```
R2# EIGRP: pkt key id = 1, authentication mismatch
```

You will see the message above in the debug output on R2. At least it tells us that key 1 is the one with the error.

If you want to spice it up, you can set an accept and expire lifetime on keys. The idea behind this is that you can have keys that are only valid for a day, a week, a month, or something else. Do you want to use this in real life? It might enhance security but also make maintenance a bit more complex…

Before you configure keys with a limited lifetime, make sure you set the correct time and date. You can do this manually on each router, but it’s better to use an [NTP (Network Time Protocol)](https://networklessons.com/cisco/ccnp-encor-350-401/cisco-network-time-protocol-ntp) server, so all the routers have the same time/date.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface fastEthernet 0/0
 ip address 192.168.12.1 255.255.255.0
 ip authentication mode eigrp 12 md5 
 ip authentication key-chain eigrp 12 MY_KEY_CHAIN
!
router eigrp 12
 network 192.168.12.0
!
key chain MY_KEY_CHAIN
 key 1
  key-string MY_KEY_STRING
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface fastEthernet 0/0
 ip address 192.168.12.2 255.255.255.0
 ip authentication mode eigrp 12 md5 
 ip authentication key-chain eigrp 12 MY_KEY_CHAIN
!
router eigrp 12
 network 192.168.12.0
!
key chain MY_KEY_CHAIN
 key 1
  key-string MY_KEY_STRING
!
end
```
{% endtab %}
{% endtabs %}

Anyway, that’s all I wanted to show you! If you have any more questions, please leave a comment!
