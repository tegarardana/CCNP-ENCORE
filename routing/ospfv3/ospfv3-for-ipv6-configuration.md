# OSPFv3 for IPv6 Configuration

When we use OSPF for IPv4 we are using OSPFv2. OSPF has been updated for IPv6 and is now called OSPFv3. These are two different routing protocols and in this lesson I’ll show you how to configure OSPFv3 so that you can route IPv6 traffic. Here’s the topology we’ll use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ipv6-routing-two-routers.png" alt=""><figcaption></figcaption></figure>

&#x20;Let’s start with the configuration of the interfaces and the IPv6 addresses. We don’t have to configure any global unicast IPv6 addresses on the FastEthernet interfaces because OSPFv3 uses link-local addresses for the neighbor adjacency and sending LSAs. Here’s the configuration:

<pre><code><strong>R1(config)#ipv6 unicast-routing 
</strong><strong>R1(config)#interface loopback 0
</strong><strong>R1(config-if)#ipv6 address 2001::1/128
</strong></code></pre>

<pre><code><strong>R2(config)#ipv6 unicast-routing 
</strong><strong>R2(config)#interface loopback 0
</strong><strong>R2(config-if)#ipv6 address 2001::2/128
</strong></code></pre>

Don’t forget to enable **`IPv6 unicast routing`** otherwise, no routing protocol will work for IPv6. Let’s check the interfaces:

<pre><code><strong>R1#show ipv6 interface brief 
</strong>FastEthernet0/0            [up/up]
Loopback0                  [up/up]
    FE80::CE09:18FF:FE0E:0
    2001::1
</code></pre>

<pre><code><strong>R2#show ipv6 interface brief 
</strong>FastEthernet0/0            [up/up]
Loopback0                  [up/up]
    FE80::CE0A:18FF:FE0E:0
    2001::2
</code></pre>

After configuring the IPv6 addresses on the loopback interface, you can see the global unicast and the link-local IPv6 addresses. There is no link-local address on the FastEthernet interfaces, however, so we’ll have to fix this:

<pre><code><strong>R1(config)#interface fastEthernet 0/0
</strong><strong>R1(config-if)#ipv6 enable
</strong></code></pre>

<pre><code><strong>R2(config)#interface fastEthernet 0/0
</strong><strong>R2(config-if)#ipv6 enable
</strong></code></pre>

<pre><code><strong>R1#show ipv6 interface brief 
</strong>FastEthernet0/0            [up/up]
    FE80::CE09:18FF:FE0E:0
Loopback0                  [up/up]
    FE80::CE09:18FF:FE0E:0
    2001::1
</code></pre>

<pre><code><strong>R2#show ipv6 interface brief 
</strong>FastEthernet0/0            [up/up]
    FE80::CE0A:18FF:FE0E:0
Loopback0                  [up/up]
    FE80::CE0A:18FF:FE0E:0
    2001::2
</code></pre>

Now we can configure OSPFv3:

<pre><code><strong>R1(config)#ipv6 router ospf 1
</strong><strong>R1(config-rtr)#router-id 1.1.1.1
</strong><strong>R1(config-rtr)#exit
</strong><strong>R1(config)#interface fastEthernet 0/0
</strong><strong>R1(config-if)#ipv6 ospf 1 area 0
</strong><strong>R1(config-if)#exit
</strong><strong>R1(config)#interface loopback 0
</strong><strong>R1(config-if)#ipv6 ospf 1 area 0
</strong></code></pre>

<pre><code><strong>R2(config)#ipv6 router ospf 1
</strong><strong>R2(config-rtr)#router-id 2.2.2.2
</strong><strong>R2(config-rtr)#exit
</strong><strong>R2(config)#interface fastEthernet 0/0
</strong><strong>R2(config-if)#ipv6 ospf 1 area 0
</strong><strong>R2(config-if)#exit 
</strong><strong>R2(config)#interface loopback 0
</strong><strong>R2(config-if)#ipv6 ospf 1 area 0
</strong></code></pre>

Just like OSPFv2, you need to start a process and specify a process ID. For OSPFv3, we have to use the `ipv6 router ospf` command. Just like EIGRP for IPv6, we need a router ID if we don’t have any IPv4 addresses configured on our router. Finally, go to the interface and use the `ipv6 ospf area` command to enable OSPFv3, and select the correct area. Let’s see if we have neighbors:

<pre><code><strong>R1#show ipv6 ospf neighbor 
</strong>
Neighbor ID  Pri   State           Dead Time   Interface ID    Interface
2.2.2.2       1   FULL/BDR        00:00:30    4               FastEthernet0/0
</code></pre>

<pre><code><strong>R2#show ipv6 ospf neighbor 
</strong>
Neighbor ID   Pri   State           Dead Time   Interface ID    Interface
1.1.1.1       1   FULL/DR         00:00:39    4               FastEthernet0/0
</code></pre>

Use `show ipv6 ospf neighbor` to see your neighbors. It’s funny to see the old IPv4 neighbor ID even though OSPFv3 is IPv6-only. Let’s check the routing tables:

<pre><code><strong>R1#show ipv6 route ospf 
</strong>IPv6 Routing Table - 3 entries
Codes: C - Connected, L - Local, S - Static, R - RIP, B - BGP
       U - Per-user Static route, M - MIPv6
       I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary
       O - OSPF intra, OI - OSPF inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       D - EIGRP, EX - EIGRP external
O   2001::2/128 [110/10]
     via FE80::C00F:1AFF:FEA7:0, FastEthernet0/0
</code></pre>

<pre><code><strong>R2#show ipv6 route ospf 
</strong>IPv6 Routing Table - 3 entries
Codes: C - Connected, L - Local, S - Static, R - RIP, B - BGP
       U - Per-user Static route, M - MIPv6
       I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary
       O - OSPF intra, OI - OSPF inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       D - EIGRP, EX - EIGRP external
O   2001::1/128 [110/10]
     via FE80::C00E:1AFF:FEA7:0, FastEthernet0/0
</code></pre>

In our routing table, we find the fresh OSPFv3 route. That’s it! This is a fairly simple example, but it should help you to get going with OSPFv3 for IPv6.

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
interface loopback 0
 ipv6 address 2001::1/128
 ipv6 ospf 1 area 0
!
interface fastEthernet 0/0
 ipv6 enable
 ipv6 ospf 1 area 0
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
interface loopback 0
 ipv6 address 2001::2/128
 ipv6 ospf 1 area 0
!
interface fastEthernet 0/0
 ipv6 enable
 ipv6 ospf 1 area 0
!
ipv6 router ospf 1
 router-id 2.2.2.2
!
end
```
{% endtab %}
{% endtabs %}

If you have any questions, please leave a comment.
