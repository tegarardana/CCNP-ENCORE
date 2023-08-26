# OSPFv3 for IPv4 Configuration

OSPF has seen quite some changes since it was introduced somewhere in the 1980s.

The first time it was documented was in 1989 in RFC 1131. Some improvements were made in OSPF version 2, first announced in RFC 1247, updated by RFC 1583, 2178 and 2328.

Later it was updated so it could support IPv6, this resulted in OSPFv3 which was described in RFC 2740, updated by RFC 5340.

Long story short…we used OSPF version 2 of IPv4 and OSPF version 3 for IPv6.

The IETF kept updating OSPF version 3 and since RFC 5838 it supports address families (just like BGP). This means we don’t have to run OSPFv2 and OSPFv3 next to each other, one routing instance supports IPv4 and IPv6 at the same time.

In this lesson, I’ll explain how to configure OSPFv3 for IPv4.

## Configuration

OSPFv3 with address family support has been added since IOS 15.1(3)S and 15.2(1)T. To demonstrate this I’ll use two routers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/ospf-two-routers-single-area.png" alt=""><figcaption></figcaption></figure>

Let’s enable OSPFv3 on our routers:

<pre><code><strong>R1(config)#router ospfv3 1
</strong>%OSPFv3: IPv6 routing not enabled
</code></pre>

Even though I only want to configure routing for IPv4, OSPFv3 still uses IPv6 link-local addresses so we have to to enable IPv6:

<pre><code>R1 &#x26; R2
<strong>(config)#ipv6 unicast-routing
</strong></code></pre>

Now we’ll try to enable OSPFv3 on the interface:

<pre><code>R1 &#x26; R2
<strong>(config)#interface GigabitEthernet 3
</strong><strong>(config-if)#ospfv3 1 ipv4 area 0
</strong>
% OSPFv3: IPV6 is not enabled on this interface
</code></pre>

If you don’t have IPv6 enable on your interfaces, you get the above error message. Let’s enable it:

<pre><code>R1 &#x26; R2
<strong>(config)#interface GigabitEthernet 3
</strong><strong>(config-if)#ipv6 enable
</strong><strong>(config-if)#ospfv3 1 ipv4 area 0
</strong></code></pre>

Once you enable IPv6 on the interface, a link-local address is created. The routers can now establish a neighbor adjacency. Let’s see if we can advertise something in OSPFv3:

<pre><code><strong>R2(config)#interface loopback 0
</strong><strong>R2(config)#ip address 2.2.2.2 255.255.255.0
</strong><strong>R2(config-if)#ospfv3 1 ipv4 area 0       
</strong>     
% OSPFv3: IPV6 is not enabled on this interface

<strong>R2(config-if)#ipv6 enable 
</strong><strong>R2(config-if)#ospfv3 1 ipv4 area 0
</strong></code></pre>

Above you can see I created a new loopback interface with an IP address, once I try to advertise it I still get en error that it requires an IPv6 address. This is a bit awkward since I won’t be using this interface to establish neighbor adjacencies, it’s only an IPv4 network that I want to advertise. Anyway, we enable IPv6 and then we can advertise it. Let’s verify our work:

## Verification

First let’s check if we have neighbors:

<pre><code><strong>R1#show ospfv3 neighbor 
</strong>
          OSPFv3 1 address-family ipv4 (router-id 192.168.12.1)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
192.168.12.2      1   FULL/DR         00:00:35    8               GigabitEthernet3
</code></pre>

The output is the same as “show ip ospf neighbor” but now we use another command. Same thing applies to looking at the OSPF LSDB:

<pre><code><strong>R1#show ospfv3 database 
</strong>
          OSPFv3 1 address-family ipv4 (router-id 192.168.12.1)

		Router Link States (Area 0)

ADV Router       Age         Seq#        Fragment ID  Link count  Bits
 192.168.12.1    154         0x80000002  0            1           None
 192.168.12.2    155         0x80000002  0            1           None

		Net Link States (Area 0)

ADV Router       Age         Seq#        Link ID    Rtr count
 192.168.12.2    155         0x80000001  8          2

		Link (Type-8) Link States (Area 0)

ADV Router       Age         Seq#        Link ID    Interface
 192.168.12.1    198         0x80000001  8          Gi3
 192.168.12.2    195         0x80000001  8          Gi3

		Intra Area Prefix Link States (Area 0)

ADV Router       Age         Seq#        Link ID    Ref-lstype  Ref-LSID
 192.168.12.2    105         0x80000001  0          0x2001      0
 192.168.12.2    155         0x80000001  8192       0x2002      8
</code></pre>

If you want to look at the OSPF entries in the routing table then the “old” command doesn’t work:

<pre><code><strong>R1#show ip route ospf
</strong></code></pre>

This will give you no routes, the following command will work though:

<pre><code><strong>R1#show ip route ospfv3 
</strong>Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/1] via 192.168.12.2, 00:02:44, GigabitEthernet3
</code></pre>

Now we see the network we learned from R2. That’s all there is to it! I hope this has been useful, if you have any questions feel free to leave a comment.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ipv6 unicast-routing
ipv6 cef
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.0
 ipv6 enable
 ospfv3 1 ipv4 area 0
!
interface GigabitEthernet3
 ip address 192.168.12.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
 ipv6 enable
 ospfv3 1 ipv4 area 0
!
router ospfv3 1
 !
 address-family ipv4 unicast
 exit-address-family
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ipv6 unicast-routing
ipv6 cef
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.0
 ipv6 enable
 ospfv3 1 ipv4 area 0
!
interface GigabitEthernet3
 ip address 192.168.12.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
 ipv6 enable
 ospfv3 1 ipv4 area 0
!
router ospfv3 1
 !
 address-family ipv4 unicast
 exit-address-family
!
end
```
{% endtab %}
{% endtabs %}
