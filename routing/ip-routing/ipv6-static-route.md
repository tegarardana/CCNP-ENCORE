# IPv6 Static Route

If you know how to [configure a static route for IPv4](https://networklessons.com/cisco/ccnp-encor-350-401/how-to-configure-static-route-on-cisco-ios-router), you shouldn’t have any issues with IPv6 static routes. The configuration and syntax are similar. There are only some minor differences. In this lesson, I will show you how to configure all IPv6 static route types.

## Configuration

To demonstrate this topology, I will use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/r1-r2-serial-link-ipv6-addresses.png" alt=""><figcaption></figcaption></figure>

R1 and R2 are connected with a serial link. R2 has a loopback interface with IPv6 address 2001:DB8:2:2::2/64. Let’s see if we can reach this address.

### Static route for a prefix

Let’s start with a simple example where we create a static route for the prefix we want to reach: 2001:DB8:2:2::/64.

#### **Static route for a prefix – outgoing interface**

Like with IPv4, it is possible to use an interface as the next hop. This will only work with _point-to-point_ interfaces:

<pre><code><strong>R1(config)#ipv6 route 2001:DB8:2:2::/64 Serial 0/0/0
</strong></code></pre>

Here’s what the routing table looks like:

<pre><code><strong>R1#show ipv6 route static
</strong>
S   2001:DB8:2:2::/64 [1/0]
     via Serial0/0/0, directly connected
</code></pre>

Let’s see if it works:

<pre><code><strong>R1#ping 2001:DB8:2:2::2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:2:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/4 ms
</code></pre>

Our ping is working.

If you try this with a FastEthernet interface, you’ll see that the router will accept the command, but the ping won’t work. You can’t use this for multi-access interfaces.

#### **Static route for a prefix – global unicast next hop**

Instead of an outgoing interface, we can also specify the global unicast address as the next hop:

<pre><code><strong>R1(config)#ipv6 route 2001:DB8:2:2::/64 2001:DB8:12:12::2
</strong></code></pre>

Here’s what the routing table looks like:

<pre><code><strong>R1#show ipv6 route static
</strong>
S   2001:DB8:2:2::/64 [1/0]
     via 2001:DB8:12:12::2
</code></pre>

Let’s see if it works:

<pre><code><strong>R1#ping 2001:DB8:2:2::2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:2:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/4 ms
</code></pre>

No problem at all…

Instead of global unicast addresses, you can also use unique local addresses. These are the IPv6 equivalent of IPv4 private addresses.

#### **Static route for a prefix – link-local next hop**

One of the differences between IPv4 and IPv6 is that IPv6 generates a link-local address for each interface. These link-local addresses are also used by routing protocols like RIPng, EIGRP, OSPFv3, etc, as the next-hop addresses. Let’s see what the link-local address is of R2:

<pre><code><strong>R2#show ipv6 interface Serial 0/0/0 | include link-local
</strong>  IPv6 is enabled, link-local address is FE80::21C:F6FF:FE11:41F0 
</code></pre>

Let’s use this as the next-hop address. When you use a global unicast address as the next hop, your router can look at the routing table and figure out what outgoing interface to use to reach this global unicast address. With link-local addresses, the router has no clue which outgoing interface to use so you will have to specify both the outgoing interface and the link-local address:

<pre><code><strong>R1(config)#ipv6 route 2001:DB8:2:2::/64 Serial 0/0/0 FE80::21C:F6FF:FE11:41F0
</strong></code></pre>

Here’s what the routing table looks like:

<pre><code><strong>R1#show ipv6 route static 
</strong>
S   2001:DB8:2:2::/64 [1/0]
     via FE80::21C:F6FF:FE11:41F0, Serial0/0/0
</code></pre>

Just to be sure, let’s try a ping:

<pre><code><strong>R1#ping 2001:DB8:2:2::2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:2:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/4 ms
</code></pre>

No problems there.

### Static default route

Just like IPv4, we can also create static default routes. A default route has only zeroes (::) and a /0 prefix length. This is the equivalent of 0.0.0.0/0 in IPv4. We can do this with an interface, global unicast, or link-local address. Let’s try all options!

#### **Static default route – outgoing interface**

Let’s start with the outgoing interface first:

<pre><code><strong>R1(config)#ipv6 route ::/0 Serial 0/0/0
</strong></code></pre>

Here’s the routing table:

<pre><code><strong>R1#show ipv6 route static 
</strong>
S   ::/0 [1/0]
     via Serial0/0/0, directly connected
</code></pre>

Let’s try a quick ping:

<pre><code><strong>R1#ping 2001:DB8:2:2::2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:2:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/4 ms
</code></pre>

#### **Static default route – global unicast next hop**

Instead of an outgoing interface, let’s try a global unicast next-hop address:

<pre><code><strong>R1(config)#ipv6 route ::/0 2001:DB8:12:12::2
</strong></code></pre>

Here’s the routing table:

<pre><code><strong>R1#show ipv6 route static
</strong>
S ::/0 [1/0]
via 2001:DB8:12:12::2
</code></pre>

Let’s try a quick ping:

<pre><code><strong>R1#ping 2001:DB8:2:2::2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:2:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/4 ms
</code></pre>

Time for the next option.

#### **Static default route – link-local next hop**

Let’s replace the global unicast next hop address with a link-local address:

<pre><code><strong>R1(config)#ipv6 route ::/0 Serial 0/0/0 FE80::21C:F6FF:FE11:41F0
</strong></code></pre>

Here’s the routing table:

<pre><code><strong>R1#show ipv6 route static 
</strong>
S   ::/0 [1/0]
     via FE80::21C:F6FF:FE11:41F0, Serial0/0/0
</code></pre>

Let’s try a quick ping:

<pre><code><strong>R1#ping 2001:DB8:2:2::2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:2:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/4 ms
</code></pre>

Our ping is working.

### Static host route

We can also create static routes for a single IPv6 address, called a **static host route**. These examples are the same as the ones you have seen before, but this time, we will create an entry for 2001:DB8:2:2::2/128, which is similar to using a /32 subnet mask in IPv4.

#### **Static host route – outgoing interface**

First, we will try the outgoing interface:

<pre><code><strong>R1(config)#ipv6 route 2001:DB8:2:2::2/128 Serial 0/0/0
</strong></code></pre>

Here is the routing table:

<pre><code><strong>R1#show ipv6 route static 
</strong>
S   2001:DB8:2:2::2/128 [1/0]
     via Serial0/0/0, directly connected
</code></pre>

Let’s try a quick ping:

<pre><code><strong>R1#ping 2001:DB8:2:2::2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:2:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/4 ms
</code></pre>

#### **Static host route – global unicast next hop**

Let’s try a global unicast address as the next hop:

<pre><code><strong>R1(config)#ipv6 route 2001:DB8:2:2::2/128 2001:DB8:12:12::2
</strong></code></pre>

Here is the routing table:

<pre><code><strong>R1#show ipv6 route static
</strong>
S   2001:DB8:2:2::2/128 [1/0]
     via 2001:DB8:12:12::2
</code></pre>

And let’s try a quick ping:

<pre><code><strong>R1#ping 2001:DB8:2:2::2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:2:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/4 ms
</code></pre>

#### **Static host route – link-local next hop**

Last but not least, a link-local address as the next hop address:

<pre><code><strong>R1(config)#ipv6 route 2001:DB8:2:2::2/128 Serial 0/0/0 FE80::21C:F6FF:FE11:41F0
</strong></code></pre>

Here’s R1’s routing table:

<pre><code><strong>R1#show ipv6 route static 
</strong>
S   2001:DB8:2:2::2/128 [1/0]
     via FE80::21C:F6FF:FE11:41F0, Serial0/0/0
</code></pre>

Let’s try another ping:

<pre><code><strong>R1#ping 2001:DB8:2:2::2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:2:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/4 ms
</code></pre>

### Static floating route

We can also configure [floating static routes](https://networklessons.com/cisco/ccnp-encor-350-401/floating-static-route). To test this, I have to add another router:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/r1-r2-r3-serial-fastethernet-ipv6-addresses.png" alt=""><figcaption></figcaption></figure>

R3 is added to our topology, and I configured the same loopback address (2001:DB8:23:23::23/128) on both routers. R3 will be used as our main path to reach this address. When the link is down, we want to use R2.

Here’s the static route that is used to use R3 as the primary path:

<pre><code><strong>R1(config)#ipv6 route 2001:DB8:23:23::/64 2001:DB8:13:13::3
</strong></code></pre>

#### **Static floating route – outgoing interface**

Let’s try the outgoing interface first. The static route looks like this:

<pre><code><strong>R1(config)#ipv6 route 2001:DB8:23:23::/64 Serial 0/0/0 2
</strong></code></pre>

Note that at the end of the line above, I specified the administrative distance with a value of 2. With both interfaces up, R1 will send all traffic to R3:

<pre><code><strong>R1#show ipv6 route static 
</strong>
S   2001:DB8:23:23::/64 [1/0]
     via 2001:DB8:13:13::3
</code></pre>

Above, you can see that the default administrative distance is 1. Let’s shut the FastEthernet 0/0 interface to test our static floating route:

<pre><code><strong>R1(config)#interface FastEthernet 0/0
</strong><strong>R1(config-if)#shutdown
</strong></code></pre>

Let’s look at the routing table again:

<pre><code><strong>R1#show ipv6 route static 
</strong>
S   2001:DB8:2:2::/64 [2/0]
     via Serial0/0/0, directly connected
</code></pre>

The entry to R2 is now installed. You can also see the administrative distance value of two in the routing table.

#### **Static floating route – global unicast next hop**

Instead of the outgoing interface, we can also use a global unicast address as the next hop:

<pre><code><strong>R1(config)#ipv6 route 2001:DB8:2:2::/64 2001:DB8:12:12::2 2
</strong></code></pre>

The routing table will then look like this:

<pre><code><strong>R1#show ipv6 route static 
</strong>
S   2001:DB8:2:2::/64 [2/0]
     via 2001:DB8:12:12::2
</code></pre>

#### **Static floating route – link-local next hop**

Or use a link-local address as the next hop:

<pre><code><strong>R1(config)#ipv6 route 2001:DB8:2:2::/64 Serial 0/0/0 FE80::21C:F6FF:FE11:41F0 2
</strong></code></pre>

Here is the routing table:

<pre><code><strong>R1#show ipv6 route static 
</strong>
S   2001:DB8:2:2::/64 [2/0]
     via FE80::21C:F6FF:FE11:41F0, Serial0/0/0
</code></pre>

## Conclusion

You have now learned how to configure the following IPv6 static routes:

* Static route for a prefix
* Static default route
* Static host route
* Static floating route

And how to do this with different next-hop types:

* Outgoing interface (only for point-to-point interfaces)
* Global unicast address
* Link-local address

I hope these examples have been useful to you!
