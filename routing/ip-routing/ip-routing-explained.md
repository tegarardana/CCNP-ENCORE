# IP Routing Explained

The actual forwarding of IP packets by routers is called _IP routing_. This has nothing to do with the “learning” of network routes through static or dynamic routing protocols but has everything to do with the steps that routers have to take when they forward an IP packet from one interface to another.

In this lesson, I will walk you through an example and show you all steps that occur.

To do this, I will use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/two-hosts-two-routers-ip-mac-addresses.png" alt=""><figcaption></figcaption></figure>

Above we have two host computers and two routers. H1 is going to send an IP packet to H2 which has to be routed by R1 and R2.

## IP Routing Process

Let’s look at this step-by-step, device-by-device.

### H1

Let’s start with H1. This host creates an IP packet with its own IP address (192.168.1.1) as the source and H2 (192.168.2.2) as the destination. The _first question_ that H1 will ask itself is:

* Is the destination _local or remote?_

It answers this question by looking at its own IP address, its subnet mask and the destination IP address:

<pre><code><strong>C:\Users\H1>ipconfig
</strong>
Windows IP Configuration


Ethernet adapter Ethernet 1:

   Connection-specific DNS Suffix  . : nwl.local
   Link-local IPv6 Address . . . . . : fe80::88fd:962a:44d6:3a1f%4
   IPv4 Address. . . . . . . . . . . : 192.168.1.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.1.254
</code></pre>

H1 is in network 192.168.1.0/24 so all IP addresses in the 192.168.1.1 – 254 range are local.  Our destination (192.168.2.2) is outside of the local subnet so that means we have to use the default gateway.

H1 will now build an Ethernet frame, enters its own source MAC address and asks itself the _second question_, do I know the destination MAC address of the default gateway?

It checks its [ARP table](https://networklessons.com/cisco/ccna-routing-switching-icnd1-100-105/arp-address-resolution-protocol-explained) to find the answer:

<pre><code><strong>C:\Users\H1>arp -a
</strong>
Interface: 192.168.1.1 --- 0x4
  Internet Address      Physical Address      Type
  192.168.1.254         fa-16-3e-3f-fd-3c     dynamic
  192.168.1.255         ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.251           01-00-5e-00-00-fb     static
  224.0.0.252           01-00-5e-00-00-fc     static
  239.255.255.250       01-00-5e-7f-ff-fa     static
</code></pre>

H1 has an ARP entry for 192.168.1.254. If not, it would have sent an ARP request. We now have an Ethernet frame that carries an IP packet with the following addresses:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/ethernet-frame-h1-r1.png" alt=""><figcaption></figcaption></figure>

The frame will be on its way to R1.

### R1

This Ethernet frame makes it to R1 which has more work to do than our host.  The first thing it does, is check if the FCS (Frame Check Sequence) of the Ethernet frame is correct or not:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/ethernet-frame-fcs.png" alt=""><figcaption></figcaption></figure>

If the FCS is incorrect, the frame is dropped right away. There is no error recovery for Ethernet, this is something that is done by protocols on upper layers, like [TCP ](https://networklessons.com/cisco/ccna-routing-switching-icnd1-100-105/introduction-to-tcp-and-udp)on the transport layer.

If the FCS is correct, we will process the frame if:

* The destination MAC address is the address of the interface of the router.
* The destination MAC address is a broadcast address of the subnet that the router interface is connected to.
* The destination MAC address is a multicast address that the router listens to.

In this case, the destination MAC address matches the MAC address of R1’s GigabitEthernet 0/1 interface so we will process it. We de-encapsulate (extract) the IP packet out of the Ethernet frame which is then discarded:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/extract-ip-packet-from-ethernet-frame.png" alt=""><figcaption></figcaption></figure>

The router will now look at the IP packet, and the first thing it does is check if the header checksum is OK:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/ip-header-checksum-field.png" alt=""><figcaption></figcaption></figure>

If the header checksum is not correct, the IP packet is dropped right away. There is also no error recovery on the network layer, we rely on upper layers for this. If the header checksum is correct, we continue by looking at the destination IP address:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/ip-header-ip-destination-field.png" alt=""><figcaption></figcaption></figure>

R1 now checks its routing table to see if there is a match:

<pre><code><strong>R1#show ip route
</strong>Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/1
L        192.168.1.254/32 is directly connected, GigabitEthernet0/1
S     192.168.2.0/24 [1/0] via 192.168.12.2
      192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, GigabitEthernet0/2
L        192.168.12.1/32 is directly connected, GigabitEthernet0/2
</code></pre>

Above you can see that R1 knows how to reach the 192.168.2.0/24 network, the next hop IP address is 192.168.12.2. It will now do a second routing table lookup to see if it knows how to reach 192.168.12.2, we call this **recursive routing**. As you can see, there is an entry for 192.168.12.0/24 with GigabitEthernet 0/2 as the interface to use.

There is one thing left to do with the IP packet before we can forward it. Since we are routing it, we have to decrease the TTL (Time to Live) field by one. R1 will do this and since this changes the IP header, we have to calculate a new header checksum.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/ip-header-decreased-ttl.png" alt=""><figcaption></figcaption></figure>

Once this is done, R1 checks its ARP table to see if there is an entry for 192.168.12.2:

<pre><code><strong>R1#show ip arp
</strong>Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.1.1            58   fa16.3e87.9c2a  ARPA   GigabitEthernet0/1
Internet  192.168.1.254           -   fa16.3e3f.fd3c  ARPA   GigabitEthernet0/1
Internet  192.168.12.1            -   fa16.3e02.83a1  ARPA   GigabitEthernet0/2
Internet  192.168.12.2           95   fa16.3e01.0c98  ARPA   GigabitEthernet0/2
</code></pre>

No problem there, we have an entry in the ARP table. If not, R1 will send an ARP request to find the MAC address of 192.168.12.2. R1 builds a new Ethernet frame with its own MAC address of the GigabitEthernet 0/2 interface and R2 as the destination. The IP packet is then encapsulated in this new Ethernet frame.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/ethernet-frame-r1-r2.png" alt=""><figcaption></figcaption></figure>

And the frame will be on its way towards R2.

### R2

This Ethernet frame makes it to R2. Like R1 it will first do this:

* Check the FCS of the Ethernet frame.
* De-encapsulates the IP packet, discard the frame.
* Check the IP header checksum.
* Check the destination IP address.

In the routing table, we find this:

<pre><code><strong>R2#show ip route
</strong>Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

S     192.168.1.0/24 [1/0] via 192.168.12.1
      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/24 is directly connected, GigabitEthernet0/1
L        192.168.2.254/32 is directly connected, GigabitEthernet0/1
      192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, GigabitEthernet0/2
L        192.168.12.2/32 is directly connected, GigabitEthernet0/2
</code></pre>

Network 192.168.2.0/24 is directly connected to R2 on its GigabitEthernet 0/1 interface. R2 will now reduce the TTL of the IP packet from 254 to 253, recalculate the IP header checksum and checks its ARP table to see if it knows how to reach 192.168.2.2:

<pre><code><strong>R2#show ip arp
</strong>Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.2.2           121   fa16.3e4a.f598  ARPA   GigabitEthernet0/1
Internet  192.168.2.254           -   fa16.3e3c.7da4  ARPA   GigabitEthernet0/1
Internet  192.168.12.1          111   fa16.3e02.83a1  ARPA   GigabitEthernet0/2
Internet  192.168.12.2            -   fa16.3e01.0c98  ARPA   GigabitEthernet0/2
</code></pre>

There is an ARP entry there. The new Ethernet frame is created, the IP packet encapsulated and it has the following addresses:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/frame-r2-h2.png" alt=""><figcaption></figcaption></figure>

The frame is then forwarded to H2.

### H2

H2 receives the Ethernet frame and will:

* Check the FCS
* Find its own MAC address as the destination MAC address.
* De-encapsulates the IP packet from the frame.
* Finds its own IP address as the destination in the IP packet.

H2 then looks for the protocol field to figure out what transport layer protocol we are dealing with, what happens next depends on the transport layer protocol that is used. That’s a story for another time.

## Conclusion

You have now learned how an IP packet is forwarded from one router to another, also known as IP routing.

Let’s summarize this process.

The host has a simple decision to make:

* Is the destination on the local subnet? If yes:
  * Check the ARP table to see if the destination IP address is in the table, and if it is, use the corresponding MAC address in the destination field of the Ethernet Frame.
* Is the destination on a remote subnet? If yes:
  * Check the ARP table **to see if the IP address of the default gateway is in the table**, and if it is, use the corresponding MAC address in the destination field of the Ethernet Frame.

The router has to perform a number of tasks:

* When it receives an Ethernet frame, check if the FCS (Frame Check Sequence) is correct. If not, drop the frame.
* Check if the destination address of the frame is:
  * destined to our MAC address
  * destined to a broadcast address of the subnet our interface is in.
  * destined to a multicast address that we listen to.
* De-encapsulate the IP packet from the frame, discard the Ethernet frame.
* Look for a match in the routing table for the destination IP address, figure out what the outgoing interface and optionally, the next hop IP address is.
* Decrease the TTL (Time to Live) field in the IP header, recalculate the header checksum.
* Encapsulate the IP packet in a new Ethernet frame.
* Check the ARP table for the destination IP address or next hop IP address.
* Transmit the frame.

I hope this lesson has been useful to understand IP routing. Feel free to share this post!
