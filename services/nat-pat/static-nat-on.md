# Static NAT on

Let’s look at how to configure static NAT on a Cisco router. Here’s the topology I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/static-nat-inside-outside.png" alt=""><figcaption></figcaption></figure>

Above, you see 3 routers called Host, NAT, and Web1. Imagine our host is on our LAN and the webserver is somewhere on the Internet. Our NAT router in the middle is our connection to the Internet.

There’s a cool trick on our routers that we can use. It’s possible to disable “routing” on a router that turns it into a normal host that requires a default gateway. This is very convenient because it will save you the hassle of connecting real computers/laptops to your lab. Use `no ip routing` to disable the routing capabilities:

<pre><code><strong>Host(config)#no ip routing
</strong></code></pre>

<pre><code><strong>Web1(config)#no ip routing
</strong></code></pre>

The routing table is now gone. Let me show you:

<pre><code><strong>Host#show ip route 
</strong>Default gateway is not set

Host               Gateway           Last Use    Total Uses  Interface
ICMP redirect cache is empty
</code></pre>

<pre><code><strong>Web1#show ip route 
</strong>Default gateway is not set

Host               Gateway           Last Use    Total Uses  Interface
ICMP redirect cache is empty
</code></pre>

As you can see, the routing table on the host and Web1 is gone. We’ll have to configure a default gateway on router Host and Web1, or they won’t be able to reach each other:

<pre><code><strong>Host(config)#ip default-gateway 192.168.12.2
</strong></code></pre>

<pre><code><strong>Web1(config)#ip default-gateway 192.168.23.2
</strong></code></pre>

Both routers can use router NAT as their default gateway. Let’s see if they can reach each other:

<pre><code><strong>Host#ping 192.168.23.3
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.23.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/8/12 ms
</code></pre>

Reachability is no issue, as you can see. Now let me show you a neat trick:

<pre><code><strong>Web1#debug ip packet 
</strong>IP packet debugging is on
</code></pre>

I can use `debug ip packet` to see the IP packets that I receive. Don’t do this on a production network, or you’ll be overburdened with debug messages! Now let’s send that ping again…

```
Web1#
IP: s=192.168.12.1 (FastEthernet0/0), d=192.168.23.3, len 100, rcvd 1
```

Above, you see that our router has received an IP packet with the source IP address 192.168.12.1 and the destination IP address 192.168.23.3.

```
IP: tableid=0, s=192.168.23.3 (local), d=192.168.12.1 (FastEthernet0/0), routed via RIB
```

And it will reply with an IP packet with source address 192.168.23.3 and destination address 192.168.12.1.

Now let’s configure NAT so you can see the difference:

<pre><code><strong>NAT(config)#interface fastEthernet 1/0
</strong><strong>NAT(config-if)#ip nat inside
</strong></code></pre>

<pre><code><strong>NAT(config)#interface fastEthernet 0/0
</strong><strong>NAT(config-if)#ip nat outside
</strong></code></pre>

First we’ll have to configure the inside and outside interfaces. Our host is the “LAN” side so it’s the inside. Our web server is “on the Internet” so it’s the outside of our network. Now we can configure our static NAT rule:

<pre><code><strong>NAT(config)#ip nat inside source static 192.168.12.1 192.168.23.2
</strong></code></pre>

We use the ip nat inside command to translate an inside IP address (192.168.12.1) to an outside IP address (192.168.23.2).

<pre><code><strong> NAT#show ip nat translations 
</strong>Pro Inside global      Inside local       Outside local      Outside global
--- 192.168.23.2       192.168.12.1       ---                ---
</code></pre>

You can use the `show ip nat translations` command to verify our configuration. Now let’s send another ping and see if this configuration does anything…

<pre><code><strong>Host#ping 192.168.23.3
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.23.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/8/12 ms
</code></pre>

```
Web1#
IP: s=192.168.23.2 (FastEthernet0/0), d=192.168.23.3, len 100, rcvd 1
```

See the difference? The packet that the web server receives from the host has source IP address 192.168.23.2.

```
Web1#
IP: tableid=0, s=192.168.23.3 (local), d=192.168.23.2 (FastEthernet0/0), routed via RIB
```

And when it responds the destination, IP address is 192.168.23.2.

Now we know that static NAT is working.

{% hint style="info" %}
_In the example I just showed you, our web server no longer requires a default gateway. The packets are translated to 192.168.23.2, and this network is directly connected for the web server._
{% endhint %}

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="Host" %}
```
hostname Host
!
interface FastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
no ip routing
!
ip default-gateway 192.168.12.2
!
end
```
{% endtab %}

{% tab title="NAT" %}
```
hostname NAT
!
interface FastEthernet0/0
 ip address 192.168.23.2 255.255.255.0
 ip nat outside
!
interface FastEthernet1/0
 ip address 192.168.12.2 255.255.255.0
 ip nat inside
!
ip nat inside source static 192.168.12.1 192.168.23.2
!
end
```
{% endtab %}

{% tab title="Web1" %}
```
hostname Web1
!
interface FastEthernet0/0
 ip address 192.168.23.3 255.255.255.0
!
no ip routing
!
ip default-gateway 192.168.23.2
!
end
```
{% endtab %}
{% endtabs %}

* Configurations
* Host
* NAT
* Web1

I hope this helps you to understand NAT. In another lesson, I will demonstrate dynamic NAT and PAT to you. If you enjoyed this lesson, please share it or leave a comment!
