# IP Routing

## Introduction to Routers and Routing

In this lesson, we will take a look at the difference between switches and routers and I’ll explain to you the basics of routing.

First of all…what is a router or what is routing exactly? A switch “switches” and a router “routes,” but what does this exactly mean?

We have seen switches and you have learned that they “switch” based on MAC address information. The only concern for our switch is to know when an Ethernet frame enters one of its interfaces where it should send this Ethernet frame by looking at the destination MAC address. Switches make decisions based on Data Link layer information (layer 2).

Routers have a similar task but this time we are going to look at IP packets and as you might recall IP is on the Network layer (layer 3). Routers look at the destination IP address in an IP packet and send it out to the correct interface.

Maybe you are thinking…what is the big difference here? Why don’t we use MAC addresses everywhere and switch? Why do we need to look at IP addresses and route them? Both MAC addresses and IP addresses are unique per network device. Good question, and I’m going to show you a picture to answer this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/400-computers-2-switches.png" alt=""><figcaption></figcaption></figure>

We have two switches and to each switch are 200 computers connected. Now, if all 400 computers want to communicate with each switch, has to learn 400 MAC addresses. They need to know the MAC addresses of the computers on the left and right sides.

Now think about a really large network…for example, the Internet. There are millions of devices! Would it be possible to have millions of entries in your MAC-address table? For each device on the Internet? No way! The problem with switching is that it’s not scalable; we don’t have any hierarchy, just flat 48-bit MAC addresses. Let’s look at the same example, but now we are using routers.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/400-computers-2-routers.png" alt=""><figcaption></figcaption></figure>

What we have here is our 200 computers on the left are connected to R1 and in the 192.168.1.0 /24 network. R2 has 200 computers behind it, and the network we use over there is 192.168.2.0 /24. Routers “route” based on IP information. In our example, R1 only has to know that network 192.168.2.0 /24 is behind R2. R2 only needs to know that the 192.168.1.0 /24 network is behind R1. Are you following me here?

Instead of having a MAC address table with 400 MAC addresses we now only need a single entry on each router for each other’s networks. Switches use mac address tables to forward Ethernet frames, and routers use a routing table to learn where to forward IP packets to. As soon as you take a brand new router out of the box, It will build a routing table, but the only information you’ll find is the directly connected interfaces. Let’s start with a simple example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/routing-table-1.png" alt=""><figcaption></figcaption></figure>

Above we have one router and two computers:

* H1 has IP address 192.168.1.1 and has configured IP address 192.168.1.254 as its default gateway.
* H2 has IP address 192.168.2.2 and has configured IP address 192.168.2.254 as its default gateway.
* On our router, we have configured IP address 192.168.1.254 on interface FastEthernet 0/0 and IP address 192.168.2.254 on interface FastEthernet 1/0.
* Since we also configured a subnet mask with the IP addresses, our router knows the network addresses and will store these in its routing table.

Whenever H1 wants to send something to H2, this will happen:

1. H1 sends an IP packet with destination IP address 192.168.2.2.
2. H1 checks its own IP address and subnet mask and concludes that 192.168.2.2 is in another subnet. As a result, it will forward the IP packet to its default gateway.
3. The router receives the IP packet, checks the destination IP address, and scans the routing table. The IP address 192.168.2.2 matches the 192.168.2.0 /24 entry, and the router will forward the IP packet out if its FastEthernet 1/0 interface.
4. H2 receives the IP packet, and life is good!

Are you following me so far? Let’s configure this scenario on a real router to see what it looks like. First I’ll show you the configuration of the computers:

<pre><code><strong>C:\Documents and SettingsH1>ipconfig
</strong>Windows IP Configuration
Ethernet adapter Local Area Connection:
        Connection-specific DNS Suffix  . :
        IP Address. . . . . . . . . . . . : 192.168.1.1
        Subnet Mask . . . . . . . . . . . : 255.255.255.0
        Default Gateway . . . . . . . . . : 192.168.1.254
</code></pre>

```
C:\Documents and Settings\H2>ipconfig
Windows IP Configuration 
Ethernet adapter Local Area Connection:
        Connection-specific DNS Suffix  . :
        IP Address. . . . . . . . . . . . : 192.168.2.2
        Subnet Mask . . . . . . . . . . . : 255.255.255.0
        Default Gateway . . . . . . . . . : 192.168.2.254
```

Above, you see the IP addresses and the default gateways. Let’s configure our router:

```
R1(config)#interface fastEthernet 0/0
R1(config-if)#no shutdown
R1(config-if)#ip address 192.168.1.254 255.255.255.0
R1(config-if)#exit
R1(config)#interface FastEthernet 1/0
R1(config-if)#no shutdown
Router(config-if)#ip address 192.168.2.254 255.255.255.0
```

I will configure the IP addresses on the interfaces. That’s it. Now we can check the routing table:

```
R1#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level
       ia - IS-IS inter area, * - candidate default, U - per-user static
       o - ODR, P - periodic downloaded static route
Gateway of last resort is not set
C    192.168.1.0/24 is directly connected, FastEthernet0/0
C    192.168.2.0/24 is directly connected, FastEthernet1/0
```

As you can see, the router knows about both directly connected networks.

Let’s see if we can ping from H1 to H2:

<pre><code><strong>C:\UsersH1>ping 192.168.2.2
</strong>Pinging 192.168.2.2 with 32 bytes of data:
Reply from 192.168.2.2: bytes=32 time&#x26;lt;1ms TTL=128
Reply from 192.168.2.2: bytes=32 time&#x26;lt;1ms TTL=128
Reply from 192.168.2.2: bytes=32 time&#x26;lt;1ms TTL=128
Reply from 192.168.2.2: bytes=32 time&#x26;lt;1ms TTL=128
Ping statistics for 192.168.2.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
</code></pre>

Excellent, the ping is working! We just successfully routed our first IP packet!

{% hint style="warning" %}
Be aware that when you try to ping from one Windows computer to another that your firewall might be blocking ICMP traffic.
{% endhint %}

If you are reaching some server on the Internet, you are going through many routers to reach your destination. If you want, you can see through which routers your IP packets are traveling to reach the destination. You can do this with `traceroute`. This is what it looks like if I want to reach www.cisco.com from my computer:

```
C:\Users\Computer>tracert www.cisco.com
Tracing route to e144.dscb.akamaiedge.net [95.100.128.170]
over a maximum of 30 hops: 
  1    <1 ms    <1 ms    <1 ms  192.168.154.2
  2    <1 ms    <1 ms    <1 ms  192.168.81.254
  3     9 ms     7 ms     9 ms  10.224.124.1
  4     8 ms     7 ms    10 ms  tb-rc0001-cr101-irb-201.core.as9143.net [213.51.150.129]
  5    31 ms    10 ms    13 ms  asd-lc0006-cr101-ae5-0.core.as9143.net [213.51.158.18]
  6    11 ms    12 ms    11 ms  ae1.ams10.ip4.tinet.net [77.67.64.61]
  7    11 ms    14 ms    14 ms  r22.amstnl02.nl.bb.gin.ntt.net [195.69.144.36]
  8    14 ms    15 ms    11 ms  ae-2.r03.amstnl02.nl.bb.gin.ntt.net [129.250.2.211]
  9    14 ms    11 ms    11 ms  81.20.67.150
 10    12 ms    11 ms    11 ms  95.100.128.170
Trace complete.
```

Above, you can see that I travel through 10 routers to reach www.cisco.com. You’ll see the IP addresses of the routers, and my computer also did a hostname lookup, so you’ll see the router names. Traceroute uses the ICMP protocol.

In another lesson, we will take a look at **static** and **dynamic** routing. I hope you enjoyed this lesson. If you have any questions, please leave a comment!
