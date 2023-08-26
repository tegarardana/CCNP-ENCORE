# OSPFv3 IPv6 Default Route

Just like OSPF for IPv4, it is possible to advertise a default route in OSPFv3 for IPv6. In this lesson, I’ll show you how to do this.

## Configuration

We only need two routers for this example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/03/ipv6-r1-r2-loopback-l0.png" alt=""><figcaption></figcaption></figure>

R2 has a loopback interface with IPv6 address 2001:DB8:2:2::2/128. We won’t advertise this in OSPFv3 directly but will reach it from R1 with a default route that is advertised by R2.

First, we have to enable IPv6 routing:

<pre><code>R1 &#x26; R2
<strong>(config)#ipv6 unicast-routing 
</strong></code></pre>

Let’s configure some global unicast IPv6 addresses. We don’t need global unicast addresses for OSPFv3 but we will need them if we want to send a ping from R1 to R2’s loopback address.

<pre><code><strong>R1(config)#interface GigabitEthernet 0/1
</strong><strong>R1(config-if)#ipv6 address 2001:DB8:12:12::1/64
</strong></code></pre>

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ipv6 address 2001:DB8:12:12::2/64
</strong>
<strong>R2(config)#interface loopback 0
</strong><strong>R2(config-if)#ipv6 address 2001:DB8:2:2::2/128
</strong></code></pre>

Let’s enable OSPFv3 on R1:

<pre><code><strong>R1(config)#ipv6 router ospf 1
</strong><strong>R1(config-rtr)#router-id 1.1.1.1
</strong>
<strong>R1(config)#interface GigabitEthernet 0/1
</strong><strong>R1(config-if)#ipv6 ospf 1 area 0
</strong></code></pre>

We do the same thing on R2, but also include the default route:

<pre><code><strong>R2(config)#ipv6 router ospf 1
</strong><strong>R2(config-rtr)#router-id 2.2.2.2
</strong><strong>R2(config-rtr)#default-information originate always
</strong>
<strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ipv6 ospf 1 area 0
</strong></code></pre>

The default-information originate command is what advertises the default route, it’s the same command that OSPFv2 for IPv4 uses.

{% hint style="warning" %}
The **always** parameter is **required** if you don’t have a default route in your own local routing table. If R2 had a default route pointing to another router, then you can remove the always parameter.
{% endhint %}

## Verification

Let’s verify our work. First, let’s make sure our two routers are OSPFv3 neighbors:

<pre><code><strong>R1#show ipv6 ospf neighbor 
</strong>
            OSPFv3 Router with ID (1.1.1.1) (Process ID 1)

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
2.2.2.2           1   FULL/BDR        00:00:34    3               GigabitEthernet0/1
</code></pre>

This seems to be the case. Let’s check if R1 has learned a default route from R2:

<pre><code><strong>R1#show ipv6 route ospf 
</strong>
OE2 ::/0 [110/1], tag 1
     via FE80::F816:3EFF:FE06:2CB2, GigabitEthernet0/1
</code></pre>

Above you can see the default route. Note that it is advertised as an OSPF external type 2 route with a default cost of 1. Let’s see if we can ping the loopback interface of R2:

<pre><code><strong>R1#ping 2001:DB8:2:2::2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:2:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/6/18 ms
</code></pre>

Our ping is working, which proves that our default route works.

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
interface GigabitEthernet0/1
 no ip address
 ipv6 address 2001:DB8:12:12::1/64
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
ipv6 cef
!
interface Loopback0
 no ip address
 ipv6 address 2001:DB8:2:2::2/128
!
interface GigabitEthernet0/1
 no ip address
 ipv6 address 2001:DB8:12:12::2/64
 ipv6 ospf 1 area 0
!
ipv6 router ospf 1
 router-id 2.2.2.2
 default-information originate always
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

You have now learned how to configure an IPv6 OSPFv3 default route with the default-information originate command. Keep in mind you need the always parameter if you don’t have a default route in the routing table of the router that is going to advertise the default route. The default route type is an external type 2 with a cost of 1.
