# NAT Virtual Interfac

The most common method to [configure NAT on Cisco IOS routers](https://networklessons.com/cisco/ccnp-encor-350-401/introduction-to-nat-and-pat) is what we call “domain-based NAT”. Each interface on the router has to be configured as “inside” or “outside”. This method of configuring NAT is considered the legacy method. The new way of configuring NAT is by using the **NAT virtual interface**.

Before I show you the configuration of the NAT virtual interface, let’s look at the differences between legacy NAT and the NAT virtual interface.

When we use the inside and outside domains then the “order of operations” of the router looks like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/03/cisco-nat-inside-outside-operations.png" alt=""><figcaption></figcaption></figure>

When an IP packet is received on the inside interface, then the router will first do a looking in the routing table and then it will perform NAT translation. When we receive IP packets on the outside interface, then it’s the other way around:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/03/cisco-nat-outside-inside-operations.png" alt=""><figcaption></figcaption></figure>

IP packets will first be NAT translated and then routed. Now you might be wondering, why should I care about this? To answer that question, we’ll have to do a little experiment. Here are three routers that we will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/03/r1-r2-r3-nat-virtual-interface.png" alt=""><figcaption></figcaption></figure>

R1 on the left side is on our LAN, the “inside” of our network. R3 is on the “outside” and R2 will be configured for NAT.

{% tabs %}
{% tab title="Configuration" %}
Want to try this example yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
no ip routing
!
no ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
no ip routing
!
no ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.23.3 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

Let’s start by defining the inside and outside interfaces and we’ll add a simple static NAT rule:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ip nat inside
</strong>
<strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#ip nat outside
</strong>
<strong>R2(config)#ip nat inside source static 192.168.12.1 192.168.23.1
</strong></code></pre>

The NAT rule above will translate source IP address 192.168.12.1 (R1) to 192.168.23.1. We’ll see if it works by sending a ping from R1 to R3. Before we do this, we’ll have to add a default route on R1 so that it knows how to reach 192.168.23.3 (R3):

<pre><code><strong>R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.12.2
</strong></code></pre>

We want to see the routing and NAT actions of R2 so let’s enable some debugs:

<pre><code><strong>R2#debug ip packet detail 
</strong>IP packet debugging is on (detailed)

<strong>R2#debug ip nat detailed 
</strong>IP NAT detailed debugging is on
</code></pre>

If we want to see all debug info, then we should disable route caching on R2 so that packets are handled by the CPU:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#no ip route-cache 
</strong>
<strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#no ip route-cache 
</strong></code></pre>

Now let’s send that ping from R1 to R3:

<pre><code><strong>R1#ping 192.168.23.3 repeat 1
</strong>Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 192.168.23.3, timeout is 2 seconds:
!
Success rate is 100 percent (1/1), round-trip min/avg/max = 42/42/42 ms
</code></pre>

The ping is successful. Let’s take a closer look at R2:

```
R2#
IP: tableid=0, s=192.168.12.1 (GigabitEthernet0/1), d=192.168.23.3 (GigabitEthernet0/2), routed via FIB
NAT: i: icmp (192.168.12.1, 2) -> (192.168.23.3, 2) [6]     
NAT: s=192.168.12.1->192.168.23.1, d=192.168.23.3 [6]
IP: s=192.168.23.1 (GigabitEthernet0/1), d=192.168.23.3 (GigabitEthernet0/2), g=192.168.23.3, len 100, forward
    ICMP type=8, code=0
```

What we see above is that the IP packet from 192.168.12.1 to 192.168.23.3 is routed first and then translated by NAT. What about the return traffic?

```
R2#
NAT*: o: icmp (192.168.23.3, 2) -> (192.168.23.1, 2) [6]
NAT*: s=192.168.23.3, d=192.168.23.1->192.168.12.1 [6]
IP: tableid=0, s=192.168.23.3 (GigabitEthernet0/2), d=192.168.12.1 (GigabitEthernet0/1), routed via FIB
IP: s=192.168.23.3 (GigabitEthernet0/2), d=192.168.12.1 (GigabitEthernet0/1), g=192.168.12.1, len 100, forward
    ICMP type=0, code=0
```

The return traffic is the opposite. First, it is translated by NAT and then we forward it. So far, so good.

Let’s try something else. We want our network to be as transparent as possible. R1 is currently using a default route to reach 192.168.23.3.

Let’s change this; we’ll add another NAT rule:

<pre><code><strong>R2(config)#ip nat outside source static 192.168.23.3 192.168.12.3
</strong></code></pre>

The rule above will translate 192.168.23.3 to 192.168.12.3. R1 should now be able to send a ping to 192.168.12.3 instead of 192.168.23.3. Since 192.168.12.3 is on the local subnet of R1, we won’t need its default route anymore:

<pre><code><strong>R1(config)#no ip route 0.0.0.0 0.0.0.0 192.168.12.2
</strong></code></pre>

Let’s send another ping:

<pre><code><strong>R1#ping 192.168.12.3 repeat 1
</strong>Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 192.168.12.3, timeout is 2 seconds:
!
Success rate is 100 percent (1/1), round-trip min/avg/max = 48/48/48 ms
</code></pre>

Our ping is successful but something funny is going on. Take a look at the debug of R2:

```
R2#
IP: tableid=0, s=192.168.12.1 (GigabitEthernet0/1), d=192.168.12.3 (GigabitEthernet0/1), routed via RIB
IP: s=192.168.12.1 (GigabitEthernet0/1), d=192.168.12.3 (GigabitEthernet0/1), len 100, rcvd 3
    ICMP type=8, code=0
IP: tableid=0, s=192.168.12.3 (local), d=192.168.12.1 (GigabitEthernet0/1), routed via FIB
IP: s=192.168.12.3 (local), d=192.168.12.1 (GigabitEthernet0/1), len 100, sending
    ICMP type=0, code=0
```

Above you can see that R2 is receiving the packet from 192.168.12.1 to 192.168.12.3 and replying with an IP packet from 192.168.12.3 to 192.168.12.1. There are no NAT entries at all. What is going on?

Once we configured the second NAT rule, R2 will respond to packets that are destined for 192.168.12.3. Since our traffic is going from the inside to the outside, we do routing first and NAT second.

R2 receives the packet from R1, checks its routing table and routes the packet out of the same interface. No NAT translation will occur since our traffic is not routing out of the outside interface. Here you can see the entry that R2 is “listening” for the 192.168.12.3 address:

<pre><code><strong>R2#show ip cef 192.168.12.3
</strong>192.168.12.3/32, version 15, epoch 0, receive
</code></pre>

Let’s try something else. We will send a ping from R3 to R1:

<pre><code><strong>R3#ping 192.168.23.1 repeat 1
</strong>Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 192.168.23.1, timeout is 2 seconds:
.
Success rate is 0 percent (0/1)
</code></pre>

This ping is failing. Why? Let’s check R2:

```
R2#    
NAT*: o: icmp (192.168.23.3, 4) -> (192.168.23.1, 4) [4]
NAT*: s=192.168.23.3->192.168.12.3, d=192.168.23.1 [4]
IP: tableid=0, s=192.168.12.3 (GigabitEthernet0/2), d=192.168.12.1 (GigabitEthernet0/1), routed via FIB
IP: s=192.168.12.3 (GigabitEthernet0/2), d=192.168.12.1 (GigabitEthernet0/1), g=192.168.12.1, len 100, forward
    ICMP type=8, code=0
```

Above we see that R2 receives the IP packet from 192.168.23.3 destined to 192.168.23.1. Since this IP packet is received on the outside interface, we do NAT first and routing second. You can see that source address 192.168.23.3 is translated to 192.168.12.3.

R2 will then do a routing table lookup for the destination (192.168.12.1) and forwards it towards R1. When R1 receives this packet, it will reply and here’s what R2 will do with it:

```
R2#
IP: tableid=0, s=192.168.12.1 (GigabitEthernet0/1), d=192.168.12.3 (GigabitEthernet0/1), routed via RIB
IP: s=192.168.12.1 (GigabitEthernet0/1), d=192.168.12.3 (GigabitEthernet0/1), len 100, rcvd 3
    ICMP type=0, code=0
```

R2 receives the packet from 192.168.12.1 with destination 192.168.12.3. It does a lookup in the routing table, finds a matching entry for its GigabitEthernet0/1 interface and sends it out of the interface. This packet is never translated and will never make it to R3.

So how do we solve this problem with “legacy” NAT?

R2 needs to route the IP packet to the correct destination. In our case, that means a packet with destination 192.168.12.3 has to be forwarded to 192.168.23.3. One way to fix this is by adding the “add-route” parameter in our NAT command:

<pre><code><strong>R2(config)#ip nat outside source static 192.168.23.3 192.168.12.3 add-route
</strong></code></pre>

Let’s see if this makes any difference:

<pre><code><strong>R3#ping 192.168.23.1 repeat 1
</strong>
Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 192.168.23.1, timeout is 2 seconds:
!
Success rate is 100 percent (1/1), round-trip min/avg/max = 24/24/24 ms
</code></pre>

Our ping is now working. Let’s check the debug on R2:

```
R2#    
NAT*: o: icmp (192.168.23.3, 5) -> (192.168.23.1, 5) [5]
NAT*: s=192.168.12.3, d=192.168.23.1->192.168.12.1 [5]
IP: tableid=0, s=192.168.12.3 (GigabitEthernet0/2), d=192.168.12.1 (GigabitEthernet0/1), routed via FIB
IP: s=192.168.12.3 (GigabitEthernet0/2), d=192.168.12.1 (GigabitEthernet0/1), g=192.168.12.1, len 100, forward
    ICMP type=8, code=0
```

When R2 receives the IP packet on the outside interface, then it will NAT translate the packet first and then routes it. Nothing has changed here. Let’s see what happens when R2 receives the reply from R1:

```
R2#
IP: tableid=0, s=192.168.12.1 (GigabitEthernet0/1), d=192.168.12.3 (GigabitEthernet0/2), routed via RIB
NAT: i: icmp (192.168.12.1, 5) -> (192.168.12.3, 5) [5]     
NAT: s=192.168.12.1->192.168.23.1, d=192.168.12.3 [5]
NAT: s=192.168.23.1, d=192.168.12.3->192.168.23.3 [5]
IP: s=192.168.23.1 (GigabitEthernet0/1), d=192.168.23.3 (GigabitEthernet0/2), g=192.168.23.3, len 100, forward
    ICMP type=0, code=0
```

R2 receives the IP packet on the inside interface, so it is routed first. Thanks to our “add-route” command, R2 is now using GigabitEthernet0/2 as the outgoing interface. Just before the packet is forwarded, it is translated by NAT and sent out of the GigabitEthernet0/2 interface.

{% hint style="info" %}
Instead of using the “add-route” parameter, you can also use a static route: ip route 192.168.12.3 255.255.255.255 192.168.23.3. The example above doesn’t work on IOS 15.x, the “problem” with IOS 15 is that it creates host routes in the routing table which overrules our static route.
{% endhint %}

Now you know how “legacy” NAT behaves, let’s take a look at the “modern” NAT virtual interface.

## NAT Virtual Interface

The NAT virtual interface doesn’t have a difference between inside or outside interfaces. NAT will behave symmetrically:

* When a packet is received, we do a routing lookup.
* If required, the IP packet will be NAT translated.
* Before the packet is forwarded, we do another routing lookup.

Here’s how to visualize this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/03/cisco-nat-nvi-inside-outside-translation.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/03/cisco-nat-nvi-outside-inside-translation.png" alt=""><figcaption></figcaption></figure>

Since the routing lookup is now always done after translation, we don’t need a static route anymore.

Let’s take a look at this in action.First, we remove our old configuration:

<pre><code><strong>R2(config)#no ip nat inside source static 192.168.12.1 192.168.23.1 
</strong><strong>R2(config)#no ip nat outside source static 192.168.23.3 192.168.12.3 add-route
</strong>
<strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#no ip nat inside
</strong>
<strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#no ip nat outside 
</strong></code></pre>

And configure the new NAT commands:

<pre><code><strong>R2(config)#interface range GigabitEthernet 0/1 - 1
</strong><strong>R2(config-if-range)#ip nat enable
</strong></code></pre>

On the interface level, we only have to enable NAT, that’s it.

Now let’s convert our old two NAT commands into new NAT commands:

<pre><code><strong>R2(config)#ip nat source static 192.168.12.1 192.168.23.1
</strong><strong>R2(config)#ip nat source static 192.168.23.3 192.168.12.3
</strong></code></pre>

Instead of configuring the inside and outside, we only configure the IP addresses that we want to translate. Let’s try a ping from R3 so that we see the difference between legacy NAT and the NAT virtual interface:

<pre><code><strong>R3#ping 192.168.23.1 repeat 1
</strong>
Type escape sequence to abort.
Sending 1, 100-byte ICMP Echos to 192.168.23.1, timeout is 2 seconds:
!
Success rate is 100 percent (1/1), round-trip min/avg/max = 20/20/20 ms
</code></pre>

Our ping is working, here’s the debug of R2:

```
R2#
IP: tableid=0, s=192.168.23.3 (GigabitEthernet0/2), d=192.168.23.1 (GigabitEthernet0/2), routed via RIB
NAT: i: icmp (192.168.23.3, 13) -> (192.168.23.1, 13) [13]     
NAT: s=192.168.23.3->192.168.12.3, d=192.168.23.1 [13]
NAT: s=192.168.12.3, d=192.168.23.1->192.168.12.1 [13]
IP: tableid=0, s=192.168.12.3 (GigabitEthernet0/2), d=192.168.12.1 (GigabitEthernet0/1), routed via RIB
IP: s=192.168.12.3 (GigabitEthernet0/2), d=192.168.12.1 (GigabitEthernet0/1), g=192.168.12.1, len 100, forward
    ICMP type=8, code=0
```

Above we see that R2 receives the packet and makes a routing decision first. The packet is then translated, and another routing decision takes place before it is forwarded.

Here’s what happens when R2 receives the reply from R1:

```
R2#
IP: tableid=0, s=192.168.12.1 (GigabitEthernet0/1), d=192.168.12.3 (GigabitEthernet0/1), routed via RIB
NAT: i: icmp (192.168.12.1, 13) -> (192.168.12.3, 13) [13]     
NAT: s=192.168.12.1->192.168.23.1, d=192.168.12.3 [13]
NAT: s=192.168.23.1, d=192.168.12.3->192.168.23.3 [13]
IP: tableid=0, s=192.168.23.1 (GigabitEthernet0/1), d=192.168.23.3 (GigabitEthernet0/2), routed via RIB
IP: s=192.168.23.1 (GigabitEthernet0/1), d=192.168.23.3 (GigabitEthernet0/2), g=192.168.23.3, len 100, forward
    ICMP type=0, code=0
```

R2 starts with a routing lookup, then translated the IP packet and does another routing lookup just before forwarding the IP packet.

Now you know the difference between legacy NAT and the NAT virtual interface. In the remaining part of this lesson, I’ll show you how to configure the different NAT methods using the NAT virtual interface.

## Configuration

For these examples I will use the same topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/03/r1-r2-r3-nat-virtual-interface.png" alt=""><figcaption></figcaption></figure>

### Static NAT

I just showed you static NAT but let’s do one more example where we translate the IP address from R1 to 192.168.23.1, and we will take a look at some show commands. R1 will require a default route:

<pre><code><strong>R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.12.2
</strong></code></pre>

Let’s enable NAT on the interface:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ip nat enable
</strong>
<strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#ip nat enable
</strong></code></pre>

And add the static NAT command:

<pre><code><strong>R2(config)#ip nat source static 192.168.12.1 192.168.23.1
</strong></code></pre>

This will translate 192.168.12.1 to 192.168.23.1. Let’s verify our work:

<pre><code><strong>R3#ping 192.168.23.1
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.23.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/6/9 ms
</code></pre>

The ping is working. Let’s check the NAT table:

<pre><code><strong>R2#show ip nat nvi translations  
</strong>Pro Source global      Source local       Destin  local      Destin  global
--- 192.168.23.1       192.168.12.1       ---                ---
icmp 192.168.23.3:10   192.168.23.3:10    192.168.23.1:10    192.168.12.1:10
</code></pre>

Above you can see the translation in the NAT table. The show commands for NAT virtual interface are exactly the same except you have to use “show ip nat nvi” instead of “show ip nat”. Let’s take a look at the statistics:

<pre><code><strong>R2#show ip nat nvi statistics 
</strong>Total active translations: 3 (1 static, 2 dynamic; 2 extended)
NAT Enabled interfaces:
  GigabitEthernet0/1, GigabitEthernet0/2
Hits: 9  Misses: 1
CEF Translated packets: 5, CEF Punted packets: 0
Expired translations: 0
Dynamic mappings:
</code></pre>

Above we can see the number of hits and the NAT enabled interfaces.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
no ip routing
!
no ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
ip default-gateway 192.168.12.2
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 ip nat enable
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
 ip nat enable
!
ip nat source static 192.168.12.1 192.168.23.1
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.23.3 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

### Dynamic NAT

What about dynamic NAT? It’s pretty much the same as static NAT except this time we will use a pool. Let’s configure a default route on R1:

<pre><code><strong>R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.12.2
</strong></code></pre>

And enable the interfaces:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ip nat enable
</strong>
<strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#ip nat enable
</strong></code></pre>

Now we can create a pool with addresses that we want to translate to:

<pre><code><strong>R2(config)#ip nat pool PUBLIC_IP 192.168.23.100 192.168.23.200 prefix-length 24 
</strong></code></pre>

I will use the 192.168.23.100 – 192.168.23.200 range for the pool. Let’s configure an access-list that defines the hosts that should be translated:

<pre><code><strong>R2(config)#ip access-list standard LAN
</strong><strong>R2(config-std-nacl)#permit 192.168.12.0 0.0.0.255
</strong></code></pre>

We’ll go for the entire 192.168.12.0/24 subnet. The only thing left to do is to combine the pool and access-list:

<pre><code><strong>R2(config)#ip nat source list LAN pool PUBLIC_IP
</strong></code></pre>

Let’s verify our work:

<pre><code><strong>R1#ping 192.168.23.3
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.23.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/9/12 ms
</code></pre>

Our ping is working. Let’s check the translations:

<pre><code><strong>R2#show ip nat nvi translations
</strong>Pro Source global      Source local      Destin local      Destin global
icmp 192.168.23.100:1   192.168.12.1:1   192.168.23.3:1   192.168.23.3:1
--- 192.168.23.100 192.168.12.1 --- ---
</code></pre>

Above you can see that 192.168.12.1 has been translated to 192.168.23.100 when it tries to reach 192.168.23.3. Here are the statistics:

<pre><code><strong>R2#show ip nat nvi statistics 
</strong>Total active translations: 3 (0 static, 3 dynamic; 2 extended)
NAT Enabled interfaces:
  GigabitEthernet0/1, GigabitEthernet0/2
Hits: 17  Misses: 2
CEF Translated packets: 10, CEF Punted packets: 0
Expired translations: 0
Dynamic mappings:
-- Source [Id: 1] access-list LAN pool PUBLIC_IP refcount 3
 pool PUBLIC_IP: netmask 255.255.255.0
        start 192.168.23.100 end 192.168.23.200
        type generic, total addresses 101, allocated 1 (0%), misses 0
</code></pre>

Above we see the hits, interfaces, our pool and the access-list.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
no ip routing
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
ip default-gateway 192.168.12.2
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 ip nat enable
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
 ip nat enable
!
ip nat pool PUBLIC_IP 192.168.23.100 192.168.23.200 prefix-length 24
ip nat source list LAN pool PUBLIC_IP
!
ip access-list standard LAN
 permit 192.168.12.0 0.0.0.255
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.23.3 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

### PAT Overload

What about PAT overload? This is one of the most common NAT configurations. Let’s take a look:

<pre><code><strong>R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.12.2
</strong></code></pre>

Enable the interfaces:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ip nat enable
</strong>
<strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#ip nat enable
</strong></code></pre>

Let’s configure an access-list that defines the hosts that should be translated:

<pre><code><strong>R2(config)#ip access-list standard LAN
</strong><strong>R2(config-std-nacl)#permit 192.168.12.0 0.0.0.255 
</strong></code></pre>

The entire 192.168.12.0/24 subnet should be translated. Last but not least, let’s enable PAT:

<pre><code><strong>R2(config)#ip nat source list LAN interface GigabitEthernet 0/2 overload
</strong></code></pre>

Let’s give it a try:

<pre><code><strong>R1#ping 192.168.23.3
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.23.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/10/12 ms
</code></pre>

Our ping is successful, and we can see the translation in our NAT table:

<pre><code><strong>R2#show ip nat nvi translations 
</strong>Pro Source global      Source local       Destin  local      Destin  global
icmp 192.168.23.2:2    192.168.12.1:2     192.168.23.3:2     192.168.23.3:2
</code></pre>

And here are the statistics:

<pre><code><strong>R2#show ip nat nvi statistics   
</strong>Total active translations: 2 (0 static, 2 dynamic; 2 extended)
NAT Enabled interfaces:
  GigabitEthernet0/1, GigabitEthernet0/2
Hits: 17  Misses: 2
CEF Translated packets: 10, CEF Punted packets: 0
Expired translations: 0
Dynamic mappings:
-- Source [Id: 1] access-list LAN interface GigabitEthernet0/2 refcount 2
</code></pre>

That’s all there is to it.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
no ip routing
!
no ip cef
!
interface GigabitEthernet0/0
 ip address 10.255.3.36 255.255.0.0
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
ip default-gateway 192.168.12.2
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 ip nat enable
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
 ip nat enable
!
ip nat source list LAN interface GigabitEthernet0/2 overload
!
ip access-list standard LAN
 permit 192.168.12.0 0.0.0.255
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
no ip routing
!
no ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.23.3 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

### Static PAT

What about static PAT (port forwarding)? Let’s say that we want to access the HTTP server on R1 from R3 by connecting to 192.168.23.2.

First, we create a default route on R1:

<pre><code><strong>R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.12.2
</strong></code></pre>

Let’s enable HTTP server on R1 and disable it on R2:

<pre><code><strong>R1(config)#ip http server
</strong></code></pre>

<pre><code><strong>R2(config)#no ip http server
</strong></code></pre>

Enable NAT on the interfaces:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ip nat enable
</strong>
<strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#ip nat enable
</strong></code></pre>

And configure port forwarding:

<pre><code><strong>R2(config)#ip nat source static tcp 192.168.12.1 80 192.168.23.2 80
</strong></code></pre>

Whenever someone tries to connect to TCP port 80 on 192.168.23.2, then it should be forwarded to 192.168.12.1. Let’s give it a try:

<pre><code><strong>R3#telnet 192.168.23.2 80
</strong>Trying 192.168.23.2, 80 ... Open
</code></pre>

R3 is able to connect. Let’s verify our work:

<pre><code><strong>R2#show ip nat nvi translations 
</strong>Pro Source global      Source local       Destin  local      Destin  global
tcp 192.168.23.2:80    192.168.12.1:80    ---                ---
tcp 192.168.23.3:24519 192.168.23.3:24519 192.168.23.2:80    192.168.12.1:80
</code></pre>

Above we can see that the translation has occurred. Here are the statistics:

<pre><code><strong>R2#show ip nat nvi statistics
</strong>Total active translations: 2 (1 static, 1 dynamic; 2 extended)
NAT Enabled interfaces:
  GigabitEthernet0/1, GigabitEthernet0/2
Hits: 16  Misses: 3
CEF Translated packets: 7, CEF Punted packets: 0
Expired translations: 2
Dynamic mappings:
</code></pre>

We can confirm that our packets have been translated.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
no ip routing
!
no ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
ip default-gateway 192.168.12.2
!
ip http server
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 ip nat enable
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
 ip nat enable
!
no ip http server
!
ip nat source static tcp 192.168.12.1 80 192.168.23.2 80 extendable
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.23.3 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

In this lesson, you have learned that “legacy” NAT is domain based since we require an inside and outside interface. The order of operation looks like this:

* Traffic received on the inside is routed first and then translated by NAT.
* Traffic received on the outside is translated by NAT first and then routed.

The NAT virtual interface is symmetric, the order of operation looks like this:

* Traffic received on any interface is routed first.
* Traffic is translated by NAT.
* Traffic is routed based on the new destination IP address.

You have also seen different examples how to configure static NAT, dynamic NAT, PAT and static PAT using the NAT virtual interface.

I hope you enjoyed this lesson, if you have any questions feel free to leave a comment!
