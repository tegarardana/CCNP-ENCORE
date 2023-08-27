# Troubleshooting BGP Route Advertisement

Once your [BGP neighbor adjacency is up and running](https://networklessons.com/cisco/ccnp-encor-350-401/troubleshooting-bgp-neighbor-adjacency) then you can try to troubleshoot issues with route advertisements. In a [previous lesson I explained how to fix BGP neighbor adjacencies](https://networklessons.com/cisco/ccnp-encor-350-401/troubleshooting-bgp-neighbor-adjacency), this time we’ll focus on route advertisements. Let’s look at some examples!

## BGP Network Command

Let’s start with an EBGP scenario:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/bgp-r1-r2-as1-as21.png" alt=""><figcaption></figcaption></figure>

R1 and R2 are in different autonomous systems. We are trying to advertise network 1.1.1.0 /24 from R1 to R2 but it’s not showing up on R2. Here are the configurations:

<pre><code><strong>R1#show  run | section bgp
</strong> no synchronization
 bgp log-neighbor-changes
 network 1.1.1.0
 neighbor 192.168.12.2 remote-as 2
 no auto-summary
</code></pre>

<pre><code><strong>R2#show  run | section bgp
</strong>router bgp 2
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.12.1 remote-as 1
 no auto-summary
</code></pre>

At first sight there seems to be nothing wrong here. Let’s see if R2 learned anything:

<pre><code><strong>R2#show ip bgp summary 
</strong>BGP router identifier 192.168.12.2, local AS number 2
BGP table version is 1, main routing table version 1

Neighbor     V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.1 4     1       4       4        1    0    0 00:01:26        0
</code></pre>

However R2 didn’t learn any prefixes from R1. Perhaps we have a filter?

<pre><code><strong>R1#show ip protocols | include filter
</strong>  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
</code></pre>

<pre><code><strong>R2#show ip protocols | include filter
</strong>  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
</code></pre>

Maybe there’s a distribute-list but that’s not the case here. Let’s check the network commands on R1:

<pre><code><strong>R1#show run | section router bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 network 1.1.1.0
 neighbor 192.168.12.2 remote-as 2
 no auto-summary
</code></pre>

The problem is the network command, it works differently for BGP vs our IGPs. If we configure a network command for BGP it has to be an **exact match**. In this case I forgot to add the subnet mask…let’s fix it:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

I have to make sure I type the correct subnet mask. Now check R2 again:

<pre><code><strong>R2#show ip bgp summary | begin Neighbor
</strong>Neighbor     V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.1 4     1       9       8        2    0    0 00:05:15        1
</code></pre>

<pre><code><strong>R2#show ip route bgp 
</strong>     1.0.0.0/24 is subnetted, 1 subnets
B       1.1.1.0 [20/0] via 192.168.12.1, 00:01:08
</code></pre>

Now you can see we learned the prefix and R2 installs it in the routing table…problem solved!

**Lesson learned: Type in the exact correct subnet mask…BGP is picky!**

## BGP Summarization

Let’s move onto the next scenario.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/bgp-summarization-troubleshooting.png" alt=""><figcaption></figcaption></figure>

The network engineer from AS1 wants to advertise a summary to AS 2. The network engineer from AS 2 is complaining however that he’s not receiving anything…let’s find out what is going wrong! Here are the configurations:

<pre><code><strong>R1#show run | section router bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 aggregate-address 172.16.0.0 255.255.0.0
 neighbor 192.168.12.2 remote-as 2
 no auto-summary 
</code></pre>

<pre><code><strong>R2#show run | section router bgp
</strong>router bgp 2
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.12.1 remote-as 1
 no auto-summary
</code></pre>

You can see the aggregate-address command on R1 for network 172.16.0.0 /16. Did R2 receive anything?

<pre><code><strong>R2#show ip bgp summary | begin Neighbor
</strong>Neighbor     V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.1 4     1      21      19        3    0    0 00:16:21        0
</code></pre>

Too bad…no prefixes have been received by R2. There are two things I could check here:

* See if a distribute-list is blocking prefixes inbound like I did in the previous example.
* See what R1 has in its routing table (can’t advertise what I don’t have!).

Let’s start with the routing table of R1 since I think by now you know what a distribute-list looks like..

<pre><code><strong>R1#show ip route
</strong>
C    192.168.12.0/24 is directly connected, FastEthernet0/0
     1.0.0.0/24 is subnetted, 1 subnets
C       1.1.1.0 is directly connected, Loopback0
</code></pre>

There’s nothing here that looks even close to 172.16.0.0 /16. If we want to advertise a summary we have to put something in the routing table of R1 first. Let me show you the different options:

<pre><code><strong>R1(config)#interface loopback 0
</strong><strong>R1(config-if)#ip address 172.16.0.1 255.255.255.0
</strong><strong>R1(config-if)#exit
</strong><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 172.16.0.0 mask 255.255.255.0
</strong></code></pre>

This is option 1: I’ll create a loopback interface and configure an IP address that falls within the range of the aggregate-address command. The summary can now be advertised to R2:

<pre><code><strong>R2#show ip route bgp 
</strong>     172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
B       172.16.0.0/24 [20/0] via 192.168.12.1, 00:01:25
B       172.16.0.0/16 [20/0] via 192.168.12.1, 00:01:25
</code></pre>

Now we see the summary in the routing table of R2. By default it will still advertise the other prefixes. If you don’t want this you need to use the **aggregate-address summary-only** command!

Let me show you option 2 of advertising the summary:

<pre><code><strong>R1(config)#ip route 172.16.0.0 255.255.0.0 null 0
</strong><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 172.16.0.0 mask 255.255.0.0
</strong></code></pre>

First we’ll put the 172.16.0.0 /16 network in the routing table by creating a static route and pointing it to the null0 interface. Secondly I’ll use a network command for BGP to advertise this network. The result will be this:

<pre><code><strong>R2#show ip route bgp 
</strong>B    172.16.0.0/16 [20/0] via 192.168.12.1, 00:00:45
</code></pre>

Now it shows up on R2! Problem solved!

**Lesson learned: You can’t advertise what you don’t have. Create a static route and point it to the null0 interface or create a loopback interface that has a prefix that falls within the summary address range.**

## BGP Auto-Summary

Next problem coming up, this is the topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/bgp-r1-r2-as1-as21.png" alt=""><figcaption></figcaption></figure>

Onto the next scenario. You are working as a network engineer for AS 1 and one day you get a phone call from the network engineer at AS 2 asking you why you are advertising a summary for 1.0.0.0 /8. You have no idea what the hell he is talking about so you decide to do some research. Here’s what we see on R2:

<pre><code><strong>R2#show ip route bgp
</strong>B    1.0.0.0/8 [20/0] via 192.168.12.1, 00:02:15
</code></pre>

This is what the network engineer on R2 is seeing. Let’s check why R1 is advertising this:

<pre><code><strong>R1#show ip bgp 1.0.0.0
</strong>BGP routing table entry for 1.0.0.0/8, version 3
Paths: (1 available, best #1, table Default-IP-Routing-Table)
  Advertised to update-groups:
     1         
  Local
    0.0.0.0 from 0.0.0.0 (1.1.1.1)
      Origin incomplete, metric 0, localpref 100, weight 32768, valid, sourced, best
</code></pre>

We can see that we have network 1.0.0.0 /8 in the BGP table of R1. Let’s check its routing table:

<pre><code><strong>R1#show ip route 1.0.0.0
</strong>Routing entry for 1.0.0.0/24, 1 known subnets
  Attached (1 connections)
  Redistributing via bgp 1
  Advertised by bgp 1

C       1.1.1.0 is directly connected, Loopback0
</code></pre>

Network 1.1.1.0 /24 is configured on the loopback interface but it’s in the BGP table as 1.0.0.0 /8. This could mean only 1 thing….summarization. Take a look below:

<pre><code><strong>R1#show ip protocols 
</strong>Routing Protocol is "bgp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  IGP synchronization is disabled
  Automatic route summarization is enabled
</code></pre>

A quick look at show ip protocols reveals that automatic summarization is enabled. Let’s disable it:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#no auto-summary
</strong></code></pre>

We’ll disable it on R1 so R2 learns the subnet:

<pre><code><strong>R2#show ip route bgp
</strong>     1.0.0.0/24 is subnetted, 1 subnets
B       1.1.1.0 [20/0] via 192.168.12.1, 00:00:20
</code></pre>

Now we see 1.1.1.0 /24 on R2…problem solved!

**Lesson learned: If you see classful networks in your BGP table you might have auto-summary enabled.**

{% hint style="info" %}
Some of the problems I’ve been showing you could be resolved easily by just looking and/or comparing the output of a “show run”. This might be true but keep in mind that you don’t always have access to all BGP routers in the network so maybe there’s no way to compare configurations. There could be a switch or another router in between the devices you are trying to troubleshooting that are causing issues. Using the appropriate show and debug commands will show you exactly what your router is doing and what it is advertising to other routers.
{% endhint %}

## BGP Route-Maps

Same topology, different problem:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/bgp-r1-r2-as1-as21.png" alt=""><figcaption></figcaption></figure>

The people from AS 2 are complaining that they are not receiving anything from AS 1. To keep it interesting I’m not going to show you the configurations…

<pre><code><strong>R2#show ip bgp summary | begin Neighbor
</strong>Neighbor     V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.1 4     1      51      48        1    0    0 00:08:51        0
</code></pre>

For starters, we can see that R2 is not receiving any prefixes. Do we have any filters?

<pre><code><strong>R1#show ip protocols | include filter
</strong>  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
</code></pre>

I can also verify that R1 doesn’t have any distribute-lists. Let’s check if R1 has 1.1.1.0 /24 in its BGP table:

<pre><code><strong>R1#show ip bgp 1.1.1.0
</strong>BGP routing table entry for 1.1.1.0/24, version 4
Paths: (1 available, best #1, table Default-IP-Routing-Table)
  Not advertised to any peer
  Local
    0.0.0.0 from 0.0.0.0 (1.1.1.1)
      Origin incomplete, metric 0, localpref 100, weight 32768, valid, sourced, best
</code></pre>

I can confirm that R1 does have network 1.1.1.0 /24 in its routing table so why is it not advertising this to R2?

Let’s see if R1 has configured anything special for its neighbor R2:

<pre><code><strong>R1#show ip bgp neighbors 192.168.12.2
</strong>BGP neighbor is 192.168.12.2,  remote AS 2, external link
  BGP version 4, remote router ID 192.168.12.2
  BGP state = Established, up for 00:03:34
  Last read 00:00:33, last write 00:00:33, hold time is 180, keepalive interval is 60 seconds
  Neighbor capabilities:
    Route refresh: advertised and received(old &#x26; new)
    Address family IPv4 Unicast: advertised and received
  Message statistics:
    InQ depth is 0
    OutQ depth is 0
                         Sent       Rcvd
    Opens:                 11         11
    Notifications:          0          0
    Updates:                7          0
    Keepalives:            85         86
    Route Refresh:          0          0
    Total:                103         97
  Default minimum time between advertisement runs is 30 seconds

 For address family: IPv4 Unicast
  BGP table version 3, neighbor version 3/0
 Output queue size : 0
  Index 1, Offset 0, Mask 0x2
  1 update-group member
  Outbound path policy configured
  Route map for outgoing advertisements is NEIGHBORS
                                 Sent       Rcvd
  Prefix activity:               ----       ----
    Prefixes Current:               0          0
    Prefixes Total:                 0          0
    Implicit Withdraw:              0          0
    Explicit Withdraw:              0          0
    Used as bestpath:             n/a          0
    Used as multipath:            n/a          0
</code></pre>

I will use the **show ip bgp neighbors** command to see detailed information of R2. We can see that a route-map has been applied to R2 and it’s called “NEIGHBORS”. Keep in mind that besides distribute-lists we can use also use route-maps for BGP filtering. Let’s check it out:

<pre><code><strong>R1# show route-map 
</strong>route-map NEIGHBORS, permit, sequence 10
  Match clauses:
    ip address prefix-lists: PREFIXES 
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes
</code></pre>

There’s only a match statement for prefix-list “PREFIXES”. Take a look:

<pre><code><strong>R1#show ip prefix-list 
</strong>ip prefix-list PREFIXES: 1 entries
   seq 5 deny 1.1.1.0/24
</code></pre>

There’s our troublemaker…its denying network 1.1.1.0 /24! Let’s get rid of this route-map:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#no neighbor 192.168.12.2 route-map NEIGHBORS out
</strong></code></pre>

That should take care of our problem…

<pre><code><strong>R2#show ip route bgp
</strong>     1.0.0.0/24 is subnetted, 1 subnets
B       1.1.1.0 [20/0] via 192.168.12.1, 00:00:03
</code></pre>

And finally R2 has learned about this prefix…problem solved!

**Lesson learned: Make sure there are no route-maps blocking the advertisement of prefixes.**

## IBGP Split Horizon

Here’s a new topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/bgp-as1-r1-r2-r3-ibgp.png" alt=""><figcaption></figcaption></figure>

R1 is advertising network 1.1.1.0 /24 but R3 is not learning this prefix. Here are the configurations:

<pre><code><strong>R1#show run | section router bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 network 1.1.1.0 mask 255.255.255.0
 neighbor 192.168.12.2 remote-as 1
 no auto-summary
</code></pre>

<pre><code><strong>R2#show run | section router bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.12.1 remote-as 1
 neighbor 192.168.23.3 remote-as 1
 no auto-summary
</code></pre>

<pre><code><strong>R3#show run | section router bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.23.2 remote-as 1
 no auto-summary
</code></pre>

The neighbor adjacencies have been configured,R1 is advertising network 1.1.1.0 /24. Let’s see if R2 and/or R3 have learned about it:

<pre><code><strong>R2#show ip route bgp 
</strong>     1.0.0.0/24 is subnetted, 1 subnets
B       1.1.1.0 [200/0] via 192.168.12.1, 00:00:23
</code></pre>

<pre><code><strong>R3#show ip route bgp 
</strong></code></pre>

We can see network 1.1.1.0 /24 in the routing table of R2 but it’s not showing up on R3.

Technically there is no problem. If you look closely at the BGP configuration of all three routers you can see there is only a BGP neighbor adjacency between R1 & R2 and between R2 & R3. Because of IBGP split horizon R2 does not forward network 1.1.1.0 /24 towards R3. In order to fix this we need to configure R1 and R3 to become neighbors. To accomplish this, R1 and R3 should be able to reach each other. I’ll keep it simple and use a static route for this:

<pre><code><strong>R1(config)#ip route 192.168.23.3 255.255.255.255 192.168.12.2
</strong></code></pre>

<pre><code><strong>R3(config)#ip route 192.168.12.1 255.255.255.255 192.168.23.2
</strong></code></pre>

If I’m going to configure the BGP neighbor adjacency between R1 and R3 I’ll need to make sure they can reach each other. I can use a static route or an IGP…to keep things easy I’ll use a static route this time. Now let’s configure IBGP between R1 and R3:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.23.3 remote-as 1
</strong></code></pre>

<pre><code><strong>R3(config)#router bgp 1
</strong><strong>R3(config-router)#neighbor 192.168.12.1 remote-as 1
</strong></code></pre>

That should work, take a look at R3:

<pre><code><strong>R3#show ip route bgp 
</strong>     1.0.0.0/24 is subnetted, 1 subnets
B       1.1.1.0 [200/0] via 192.168.12.1, 00:00:08
</code></pre>

And R3 has access to network 1.1.1.0 /24!

**Lesson learned: IBGP neighbor adjacencies have to be full mesh! Another solution would be by using a route-reflector or confederation.**

## BGP Next Hop

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/r1-r2-r3-as1-as2-next-hop-issue.png" alt=""><figcaption></figcaption></figure>

R3 is advertising network 3.3.3.0 /24 through EBGP and R2 installs it in the routing table. R1 however doesn’t have this network in its routing table. Here are the configurations:

<pre><code><strong>R1#show run | section router bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.12.2 remote-as 1
 no auto-summary
</code></pre>

<pre><code><strong>R2#show run | section router bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.12.1 remote-as 1
 neighbor 192.168.23.3 remote-as 2
 no auto-summary
</code></pre>

<pre><code><strong>R3#show run | section router bgp
</strong>router bgp 2
 no synchronization
 bgp log-neighbor-changes
 network 3.3.3.0 mask 255.255.255.0
 neighbor 192.168.23.2 remote-as 1
 no auto-summary
</code></pre>

Here are the configurations. To keep things easy I’m using the physical interface IP addresses to configure the BGP neighbor adjacencies. Let’s see if R2 learns about 3.3.3.0 /24:

<pre><code><strong>R2#show ip route bgp
</strong>     3.0.0.0/24 is subnetted, 1 subnets
B       3.3.3.0 [20/0] via 192.168.23.3, 00:09:37
</code></pre>

We can verify that network 3.3.3.0 /24 is in the routing table of R2. What about R1?

<pre><code><strong>R1#show ip route bgp 
</strong></code></pre>

There’s nothing in the routing table of R1 however. The first thing we should check is if it’s the BGP table or not:

<pre><code><strong>R1#show ip bgp 
</strong>BGP table version is 1, local router ID is 192.168.12.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
* i3.3.3.0/24       192.168.23.3             0    100      0 2 i
</code></pre>

We can see it’s in the BGP table and the \* indicates that this is a valid route. However I don’t see the > symbol which indicates the best path. For some reason BGP is unable to install this entry in the routing table. Take a close look at the next hop IP address (192.168.23.3). Is this IP address reachable?

<pre><code><strong>R1#show ip route 192.168.23.3 
</strong>% Network not in table
</code></pre>

R1 has no idea how to reach 192.168.23.3 so our next hop is unreachable. There are 2 ways how we can deal with this issue:

* Use a static route or routing protocol to make this next hop IP address reachable.
* Change the next hop IP address.

We’ll change the next hop IP address since I think you’ve seen enough static routes and routing protocols so far:

<pre><code><strong>R2(config)#router bgp 1
</strong><strong>R2(config-router)#neighbor 192.168.12.1 next-hop-self
</strong></code></pre>

This command will change the next hop IP address to the IP address of R2. Check out the changes on R1:

<pre><code><strong>R1#show ip bgp 
</strong>BGP table version is 2, local router ID is 192.168.12.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*>i3.3.3.0/24       192.168.12.2             0    100      0 2 i
</code></pre>

You can see the > symbol that indicates that this path has been selected as the best one. The next hop IP address is now 192.168.12.2.

<pre><code><strong>R1#show ip route bgp 
</strong>     3.0.0.0/24 is subnetted, 1 subnets
B       3.3.3.0 [200/0] via 192.168.12.2, 00:10:52
</code></pre>

Hooray! It’s in the routing table now. Are we done now? If my goal was to make this show up in the routing table then we are now finished…there’s another issue however:

<pre><code><strong>R1#ping 3.3.3.3
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
</code></pre>

My ping is unsuccessful. R1 and R2 both have network 3.3.3.0 /24 in their routing table so we know that they know where to forward the IP packets to. Let’s take a look at R3:

<pre><code><strong>R3#show ip route
</strong>
     3.0.0.0/24 is subnetted, 1 subnets
C       3.3.3.0 is directly connected, Loopback0
C    192.168.23.0/24 is directly connected, FastEthernet0/0
</code></pre>

R3 will receive an IP packet with destination 3.3.3.3 and source 192.168.12.1. You can see in the routing table that it has no idea where to send IP packets meant for 192.168.12.1. Let’s change that:

<pre><code><strong>R2(config)#router bgp 1
</strong><strong>R2(config-router)#network 192.168.12.0 mask 255.255.255.0
</strong></code></pre>

We’ll advertise network 192.168.12.0 /24 on R2. Now R3 knows how to reach it:

<pre><code><strong>R3#show ip route bgp
</strong>B    192.168.12.0/24 [20/0] via 192.168.23.2, 00:00:33
</code></pre>

Let’s try that ping from R1 again:

<pre><code><strong>R1#ping 3.3.3.3
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/11/28 ms
</code></pre>

Problem solved!

**Lesson learned: Make sure the next hop IP address is reachable so routes can be installed in the routing table and that all required networks are reachable.**

These are all the BGP “route advertisement” issues I have for you. I hope this has been useful, if you have any questions feel free to leave a comment!
