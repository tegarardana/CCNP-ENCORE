# BGP Network Advertisement

In this lesson we’ll take a look how you can advertise networks in BGP. There are two methods how we can do this:

* Network command
* Redistribution

Just like our IGPs we can use the network command to advertise something or we can redistribute networks into BGP. There’s one big difference though, the network command for BGP behaves differently.

When you use any of the IGPs (RIP, OSPF or EIGRP) then the network command is used to activate the IGP on all interfaces that fall within the range of the network command.

BGP doesn’t care about interfaces, it doesn’t even look at them. When we use the network command in BGP then BGP will only look at the routing table. When it finds the network that matches the network command, it will install it in the BGP table.

Let me show you some examples to explain what I’m talking about. We will use the following two routers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/r1-r2-as1-as2-ebgp.png" alt=""><figcaption></figcaption></figure>

R1 and R2 are in different autonomous systems so we use eBGP. Here is the BGP configuration:

<pre><code><strong>R1#show running-config | section bgp
</strong>router bgp 1
 bgp log-neighbor-changes
 neighbor 192.168.12.2 remote-as 2
</code></pre>

<pre><code><strong>R2#show running-config | section bgp
</strong>router bgp 2
 bgp log-neighbor-changes
 neighbor 192.168.12.1 remote-as 1
</code></pre>

Nothing special here, just plain eBGP between R1 and R2. Let’s advertise some networks in BGP…

## Network Command

Let’s create a loopback interface with a network and advertise it in BGP:

<pre><code><strong>R1(config)#interface loopback 1
</strong><strong>R1(config-if)#ip address 1.1.1.1 255.255.255.0
</strong>
<strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

Above we have created a loopback interface with network 1.1.1.0 /24, this is what we will advertise in BGP. Since we created a loopback interface, this network will be directly connected for R1:

<pre><code><strong>R1#show ip route 1.1.1.0
</strong>Routing entry for 1.1.1.0/24
Known via "connected", distance 0, metric 0 (connected, via interface)
Advertised by bgp 1
Routing Descriptor Blocks:
* directly connected, via Loopback1
Route metric is 0, traffic share count is 1
</code></pre>

Since it’s in the routing table, BGP will be able to install this network in the BGP table:

<pre><code><strong>R1#show ip bgp
</strong>BGP table version is 2, local router ID is 192.168.12.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       0.0.0.0                  0         32768 i
</code></pre>

Since R1 has it in its BGP table it will be able to advertise it to R2:

<pre><code><strong>R2#show ip bgp 1.1.1.1
</strong>BGP routing table entry for 1.1.1.0/24, version 2
Paths: (1 available, best #1, table default)
  Not advertised to any peer
  1
    192.168.12.1 from 192.168.12.1 (192.168.12.1)
      Origin IGP, metric 0, localpref 100, valid, external, best
</code></pre>

That’s all there is to it. Just use the network command to put the networks you want in the BGP table. One thing you have to be aware of is that you have to **use the exact network and subnet mask for the network command**. Let me give you an example:

<pre><code><strong>R1(config)#interface loopback 2
</strong><strong>R1(config-if)#ip address 11.11.11.11 255.255.255.255
</strong></code></pre>

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 11.11.11.0 mask 255.255.255.0
</strong></code></pre>

I created a loopback interface with network 11.11.11.11 /32. BGP uses the network command to advertise 11.11.11.0 /24. This network will never be placed in the BGP table since the subnet mask doesn’t match:

<pre><code><strong>R1#show ip bgp 11.11.11.11
</strong>% Network not in table
</code></pre>

Be aware of this. Make sure you type the exact network address and subnet mask when advertising something in BGP. Let’s fix this:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#no network 11.11.11.0 mask 255.255.255.0
</strong><strong>R1(config-router)#network 11.11.11.11 mask 255.255.255.255
</strong></code></pre>

With the correct network command, BGP will be able to advertise this network in the BGP table:

<pre><code><strong>R1#show ip bgp 11.11.11.11
</strong>BGP routing table entry for 11.11.11.11/32, version 5
Paths: (1 available, best #1, table default)
  Advertised to update-groups:
     1
  Local
    0.0.0.0 from 0.0.0.0 (192.168.12.1)
      Origin IGP, metric 0, localpref 100, weight 32768, valid, sourced, local, best
</code></pre>

And because R1 has it in its BGP table, R2 will be able to learn it:

<pre><code><strong>R2#show ip bgp | begin Network
</strong>   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.12.1             0             0 1 i
*> 11.11.11.11/32   192.168.12.1             0             0 1 i
</code></pre>

Alright so far so good. What if we want to advertise a network that we don’t have? Let’s say that I want to advertise network 1.0.0.0 /8 in BGP. We won’t be able to advertise this network in BGP if it’s not in the routing table. To achieve this, we’ll put this network in our routing table:

<pre><code><strong>R1(config)#ip route 1.0.0.0 255.0.0.0 null 0
</strong></code></pre>

This can be done with a static route that points to the **null interface,** everything you send to the null interface will be discarded. Using a static route like this is also called a **discard route**.

Network 1.0.0.0 /8 is now in the routing table:

<pre><code><strong>R1#show ip route 1.0.0.0
</strong>Routing entry for 1.0.0.0/8, 3 known subnets
  Attached (3 connections)
  Variably subnetted with 3 masks
S        1.0.0.0/8 is directly connected, Null0
C        1.1.1.0/24 is directly connected, Loopback1
L        1.1.1.1/32 is directly connected, Loopback1
</code></pre>

This allows BGP to advertise it:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 1.0.0.0 mask 255.0.0.0
</strong></code></pre>

Take a look at the BGP table of R1 and R2:

<pre><code><strong>R1#show ip bgp 1.0.0.0
</strong>BGP routing table entry for 1.0.0.0/8, version 6
Paths: (1 available, best #1, table default)
  Advertised to update-groups:
     1
  Local
    0.0.0.0 from 0.0.0.0 (192.168.12.1)
      Origin IGP, metric 0, localpref 100, weight 32768, valid, sourced, local, best
</code></pre>

<pre><code><strong>R2#show ip bgp 1.0.0.0
</strong>BGP routing table entry for 1.0.0.0/8, version 6
Paths: (1 available, best #1, table default)
  Not advertised to any peer
  1
    192.168.12.1 from 192.168.12.1 (192.168.12.1)
      Origin IGP, metric 0, localpref 100, valid, external, best
</code></pre>

R1 was able to install network 1.0.0.0 /8 in its BGP table and advertises it to R2.

## Redistribution

Instead of using the network command we can also redistribute something into BGP.  To demonstrate this I will create a new loopback interface, advertise it in OSPF and then redistribute it into BGP:

<pre><code><strong>R1(config)#interface loopback 3
</strong><strong>R1(config-if)#ip address 111.111.111.111 255.255.255.0
</strong><strong>R1(config-if)#exit
</strong>
<strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 111.111.111.0 0.0.0.255 area 0
</strong>
<strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#redistribute ospf 1
</strong></code></pre>

I created a loopback with network 111.111.111.0 /24, advertised it in OSPF and redistributed it into BGP. Let’s check the BGP table:

<pre><code><strong>R1#show ip bgp | begin Network
</strong>   Network          Next Hop            Metric LocPrf Weight Path
*> 1.0.0.0          0.0.0.0                  0         32768 i
*> 1.1.1.0/24       0.0.0.0                  0         32768 i
*> 11.11.11.11/32   0.0.0.0                  0         32768 i
*> 111.111.111.0/24 0.0.0.0                  0         32768 ?
</code></pre>

<pre><code><strong>R2#show ip bgp | begin Network
</strong>   Network          Next Hop            Metric LocPrf Weight Path
*> 1.0.0.0          192.168.12.1             0             0 1 i
*> 1.1.1.0/24       192.168.12.1             0             0 1 i
*> 11.11.11.11/32   192.168.12.1             0             0 1 i
*> 111.111.111.0/24 192.168.12.1             0             0 1 ?
</code></pre>

There we go, R1 placed the network in its BGP table and was able to advertise it to R2.

I hope this lesson has been useful to understand how to advertise something in BGP, if you have any question feel free to leave a comment!
