# Multi-Protocol BGP (MP-BGP)

The normal version of BGP (Border Gateway Protocol) only supported IPv4 unicast prefixes. Nowadays we use MP-BGP (Multiprotocol BGP) which supports different addresses:

* IPv4 unicast
* IPv4 multicast
* IPv6 unicast
* IPv6 multicast

MP-BGP is also used for MPLS VPN where we use MP-BGP to exchange the VPN labels. For each different “address” type, MP-BGP uses a different address family.

To allow these new addresses, MBGP has some new features that the old BGP doesn’t have:

* **Address Family Identifier (AFI):** specifies the address family.
* **Subsequent Address Family Identifier (SAFI):** Has additional information for some address families.
* **Multiprotocol Unreachable Network Layer Reachability Information (MP\_UNREACH\_NLRI):** This is an attribute used to transport networks that are unreachable.
* **BGP Capabilities Advertisement:** This is used by a BGP router to announce to the other BGP router what capabilities it supports. MP-BGP and BGP-4 are compatible, the BGP-4 router can ignore the messages that it doesn’t understand.

Since MP-BGP supports IPv4 and IPv6 we have a couple of options. MP-BGP routers can become neighbors using IPv4 addresses and exchange IPv6 prefixes or the other way around. Let’s take a look at some configuration examples…

## Configuration

### MP-BGP with IPv6 adjacency & IPv6 prefixes

Let’s start with a simple example where we use IPv6 for the neighbor adjacency and exchange some IPv6 prefixes. Here’s the topology I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/mp-bgp-r1-r2.png" alt=""><figcaption></figcaption></figure>

Here’s the configuration of R1:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 2001:db8:0:12::2 remote-as 2
</strong><strong>R1(config-router)#address-family ipv4
</strong><strong>R1(config-router-af)#no neighbor 2001:db8:0:12::2 activate
</strong><strong>R1(config-router-af)#exit
</strong><strong>R1(config-router)#address-family ipv6
</strong><strong>R1(config-router-af)#neighbor 2001:db8:0:12::2 activate
</strong><strong>R1(config-router-af)#network 2001:db8::1/128
</strong></code></pre>

In the configuration above we first specify the remote neighbor. The address-family command is used to change the IPv4 or IPv6 settings. I disable the IPv4 address-family and enabled IPv6. Last but not least, we advertised the prefix on the loopback interface. The configuration of R2 looks similar:

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 2001:db8:0:12::1 remote-as 1
</strong><strong>R2(config-router)#address-family ipv4
</strong><strong>R2(config-router-af)#no neighbor 2001:db8:0:12::1 activate
</strong><strong>R2(config-router-af)#exit
</strong><strong>R2(config-router)#address-family ipv6
</strong><strong>R2(config-router-af)#neighbor 2001:db8:0:12::1 activate
</strong><strong>R2(config-router-af)#network 2001:db8::2/128
</strong></code></pre>

After awhile the neighbor adjacency will appear:

```
R1#
%BGP-5-ADJCHANGE: neighbor 2001:DB8:0:123::2 Up
```

Now let’s check the routing tables:

<pre><code><strong>R1#show ipv6 route bgp
</strong>IPv6 Routing Table - default - 7 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary
       D - EIGRP, EX - EIGRP external, NM - NEMO, ND - Neighbor Discovery
       l - LISP
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
B   2001:DB8::2/128 [20/0]
     via FE80::217:5AFF:FEED:7AF0, FastEthernet0/0
</code></pre>

<pre><code><strong>R2#show ipv6 route bgp
</strong>IPv6 Routing Table - default - 7 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary
       D - EIGRP, EX - EIGRP external, NM - NEMO, ND - Neighbor Discovery
       l - LISP
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
B   2001:DB8::1/128 [20/0]
     via FE80::21D:A1FF:FE8B:36D0, FastEthernet0/0
</code></pre>

The routers learned each others prefixes…great! This example was pretty straight-forward but you have now learned how MP-BGP uses different address families.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface Loopback 0
 ipv6 enable
 ipv6 address 2001:DB8::1/128
!
interface fastEthernet0/0
 ipv6 enable
 ipv6 address 2001:DB8:0:12::1/64
!
router bgp 1
 bgp log-neighbor-changes
 neighbor 2001:DB8:0:12::2 remote-as 2
 !        
 address-family ipv4
  no neighbor 2001:DB8:0:12::2 activate
 exit-address-family
 !
 address-family ipv6
  network 2001:DB8::1/128
  neighbor 2001:DB8:0:12::2 activate
 exit-address-family
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface Loopback 0
 ipv6 enable
 ipv6 address 2001:DB8::2/128
!
interface fastEthernet0/0
 ipv6 enable
 ipv6 address 2001:DB8:0:12::2/64
!
router bgp 2
 bgp log-neighbor-changes
 neighbor 2001:DB8:0:12::1 remote-as 1
 !        
 address-family ipv4
  no neighbor 2001:DB8:0:12::1 activate
 exit-address-family
 !
 address-family ipv6
  network 2001:DB8::2/128
  neighbor 2001:DB8:0:12::1 activate
 exit-address-family
!
end
```
{% endtab %}
{% endtabs %}

### MP-BGP with IPv4 adjacency & IPv6 prefixes

let’s look at a more complex example, the routers will become neighbors through IPv4 but will exchange IPv6 prefixes. I’ll use the same topology but with an IPv4 subnet in between:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/mp-bgp-r1-r2-ipv4-ipv6.png" alt=""><figcaption></figcaption></figure>

Here’s the configuration:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong></code></pre>

Now we can configure the address-family for IPv6 unicast:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#address-family ipv6
</strong><strong>R1(config-router-af)#network 2001:db8::1/128
</strong><strong>R1(config-router-af)#neighbor 192.168.12.2 activate
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#address-family ipv6
</strong><strong>R2(config-router-af)#network 2001:db8::2/128
</strong><strong>R2(config-router-af)#neighbor 192.168.12.1 activate
</strong></code></pre>

Once we enter the address-family IPv6 configuration there are two things we have to configure. The prefix has to be advertised and we need to specify the neighbor. The prefixes on the loopback interface should now be advertised. Let’s check it out:

<pre><code><strong>R1#show ip bgp ipv6 unicast
</strong>BGP table version is 2, local router ID is 192.168.12.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 2001:DB8::1/128  ::                       0         32768 i
*  2001:DB8::2/128  ::FFFF:192.168.12.2
                                             0             0 2 i
</code></pre>

<pre><code><strong>R2#show ip bgp ipv6 unicast
</strong>BGP table version is 2, local router ID is 192.168.12.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  2001:DB8::1/128  ::FFFF:192.168.12.1
                                             0             0 1 i
*> 2001:DB8::2/128  ::                       0         32768 i
</code></pre>

As you can see the routers have learned about each others prefixes. There’s one problem though…we were able to exchange IPv6 prefixes but we only use IPv4 between R1 and R2, there is no valid next hop address that we can use.

To fix this, we need to use some IPv6 addresses that we can use as the next hop. We’ll have to configure a prefix between R1 and R2 for this:

<pre><code><strong>R1(config)#interface FastEthernet 0/0
</strong><strong>R1(config-if)#ipv6 address 2001:db8:0:12::1/64
</strong></code></pre>

<pre><code><strong>R2(config)#interface FastEthernet 0/0
</strong><strong>R2(config-if)#ipv6 address 2001:db8:0:12::2/64
</strong></code></pre>

Now we have IPv6 addresses that we can use as the next hop. We are using IPv4 for the neighbor peering so the next hop doesn’t change automatically. We’ll have to use a route-map for this:

<pre><code><strong>R1(config)#route-map IPV6_NEXT_HOP permit 10
</strong><strong>R1(config-route-map)#set ipv6 next-hop 2001:db8:0:12::2
</strong><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#address-family ipv6
</strong><strong>R1(config-router-af)#neighbor 192.168.12.2 route-map IPV6_NEXT_HOP in
</strong></code></pre>

<pre><code><strong>R2(config)#route-map IPV6_NEXT_HOP permit 10
</strong><strong>R2(config-route-map)#set ipv6 next-hop 2001:db8:0:12::1
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#address-family ipv6
</strong><strong>R2(config-router-af)#neighbor 192.168.12.1 route-map IPV6_NEXT_HOP in
</strong></code></pre>

Both routers now change the next hop IPv6 address of incoming prefixes. Let’s reset BGP:

<pre><code><strong>R1#clear ip bgp *
</strong></code></pre>

Take a look now:

<pre><code><strong>R1#show ip bgp ipv6 unicast | begin 2001
</strong>*> 2001:DB8::1/128  ::                       0         32768 i
*> 2001:DB8::2/128  2001:DB8:0:12::2
</code></pre>

<pre><code><strong>R2#show ip bgp ipv6 unicast | begin 2001
</strong>*> 2001:DB8::1/128  2001:DB8:0:12::1
</code></pre>

The next hop IPv6 addresses are now reachable so they can be installed in the routing table. The downside of this solution is that we had to fix the next hop ourselves, the advantage however is that we have a single BGP neighbor adjacency that can be used for the exchange of IPv4 and IPv6 prefixes.

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
interface Loopback 0
 ipv6 address 2001:DB8::1/128
!
interface fastEthernet0/0
 ipv6 enable
 ip address 192.168.12.1 255.255.255.0
 ipv6 address 2001:DB8:0:12::1/64
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 address-family ipv6
  network 2001:db8::1/128
  neighbor 192.168.12.2 activate
  neighbor 192.168.12.2 route-map IPV6_NEXT_HOP in
!
route-map IPV6_NEXT_HOP permit 10
 set ipv6 next-hop 2001:DB8:0:12::2
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
interface Loopback 0
 ipv6 address 2001:DB8::2/128
!
interface fastEthernet0/0
 ipv6 enable
 ip address 192.168.12.2 255.255.255.0
 ipv6 address 2001:DB8:0:12::2/64
!
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 address-family ipv6
  network 2001:DB8::2/128
  neighbor 192.168.12.1 activate
  neighbor 192.168.12.1 route-map IPV6_NEXT_HOP in
!
route-map IPV6_NEXT_HOP permit 10
 set ipv6 next-hop 2001:db8:0:12::1
!
end
```
{% endtab %}
{% endtabs %}
