# Troubleshooting BGP Neighbor Adjacency

BGP is a complex routing protocol and there are quite some things that could go possibly wrong. Besides being complex it’s also completely different compared to our IGPs (OSPF and EIGRP). In this lesson we’ll start with troubleshooting BGP neighbor adjacencies. Once the neighbor adjacency is working, you can focus on troubleshooting missing route advertisements.

Key to troubleshooting BGP is understanding how BGP forms a neighbor adjacency. If you are unsure how this process works exactly you might want to check my [lesson about BGP states](https://networklessons.com/cisco/ccnp-encor-350-401/bgp-neighbor-adjacency-states) first before you continue. Having said that, let’s look at some scenarios.

## BGP Interface Issues

Here’s the first topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/bgp-r1-r2-as1-as2-ebgp-1.png" alt=""><figcaption></figcaption></figure>

Two BGP routers which are connected and configured for EBGP. Unfortunately we are seeing this when check the BGP neighbor adjacency:

<pre><code><strong>R1#show ip bgp summary 
</strong>BGP router identifier 192.168.12.1, local AS number 1
BGP table version is 1, main routing table version 1

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.2    4     2       8       8        0    0    0 00:00:06 Active
</code></pre>

<pre><code><strong>R2#show ip bgp summary 
</strong>BGP router identifier 192.168.12.2, local AS number 2
BGP table version is 1, main routing table version 1

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.1    4     1       8       8        0    0    0 00:00:59 Active
</code></pre>

When two EBGP routers that are directly connected do not form a working BGP neighbor adjacency there could be a number of things that are wrong:

* Layer 2 down preventing us from reaching the other side.
* Layer 3 issue: wrong IP address on one of the routers.
* Access-list blocking TCP port 179 (BGP).
* Wrong IP address configured for BGP neighbor router.

The IP addresses that were used for the neighbor adjacency look OK so something else might be the issue. Let’s try a quick ping:

<pre><code><strong>R1#ping 192.168.12.2
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.12.2, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
</code></pre>

I can do a quick ping and I’ll see that I’m unable to reach the other side. Since layer 3 isn’t working, let’s take a look at layer 1 and 2:

<pre><code><strong>R1#show ip int brief
</strong>Interface               IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.12.1    YES manual up  up
</code></pre>

```
R2#show ip int brief
Interface               IP-Address      OK? Method Status                Protocol
FastEthernet0/0         192.168.12.2    YES manual administratively down down
```

We’ll check the interfaces and find out that someone left a shutdown command on the interface…let’s fix it:

<pre><code><strong>R2(config)#interface FastEthernet 0/0
</strong><strong>R2(config-if)#no shutdown
</strong></code></pre>

After enabling the interface you should see this:

```
R1# %BGP-5-ADJCHANGE: neighbor 192.168.12.2 Up
```

```
R2# %BGP-5-ADJCHANGE: neighbor 192.168.12.1 Up
```

That’s what we like to see. Our BGP neighbor adjacency is established…

**Lesson learned: Make sure your interfaces are up and running.**

## EBGP Requirements

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/bgp-r1-r2-as1-as2-ebgp-1.png" alt=""><figcaption></figcaption></figure>

Same topology but another issue. The goal in this scenario is to establish the EBGP neighbor adjacency between the IP addresses on the loopback interfaces.

Let me show you what the BGP configuration looks like:

<pre><code><strong>R1#show run | section bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 neighbor 2.2.2.2 remote-as 2
 no auto-summary
</code></pre>

<pre><code><strong>R2#show run | section bgp
</strong>router bgp 2
 no synchronization
 bgp log-neighbor-changes
 neighbor 1.1.1.1 remote-as 1
 no auto-summary
</code></pre>

Here’s the BGP configuration, you can see that we are using the loopback interfaces to establish a BGP neighbor adjacency. There’s no BGP neighbor adjacency:

<pre><code><strong>R1#show ip bgp summary 
</strong>BGP router identifier 192.168.12.1, local AS number 1
BGP table version is 1, main routing table version 1

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2.2.2.2         4     2       0       0        0    0    0 never    Idle
</code></pre>

<pre><code><strong>R2#show ip bgp summary 
</strong>BGP router identifier 192.168.12.2, local AS number 2
BGP table version is 1, main routing table version 1

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4     1       0       0        0    0    0 never    Idle
</code></pre>

Both routers show their BGP neighbor as idle. There are a number of things we have to check here:

* Is the IP address of the BGP neighbor reachable? We are not using the directly connected links so we might have routing issues.
* The TTL of IP packets that we use for external BGP is 1. This works for directly connected networks but if it’s not directly connected we need to change this behavior.
* By default BGP will source its updates from the IP address that is closest to the BGP neighbor. In our example that’s the FastEthernet interface. This is something we’ll have to change.

Let’s check if the IP address of the remote neighbor is reachable, take a look at the routing tables:

<pre><code><strong>R1#show ip route 
</strong>
C    192.168.12.0/24 is directly connected, FastEthernet0/0
</code></pre>

<pre><code><strong>R2#show ip route
</strong>
C    192.168.12.0/24 is directly connected, FastEthernet0/0
</code></pre>

Both routers only know about their directly connected networks. In order to reach each other’s loopback interfaces we’ll use static routing:

<pre><code><strong>R1(config)#ip route 2.2.2.2 255.255.255.255 192.168.12.2
</strong></code></pre>

<pre><code><strong>R2(config)#ip route 1.1.1.1 255.255.255.255 192.168.12.1
</strong></code></pre>

Two static routes should do the job. Let’s give it a try:

<pre><code><strong>R1#ping 2.2.2.2 source loopback 0
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2.2.2.2, timeout is 2 seconds:
Packet sent with a source address of 1.1.1.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/4/4 ms
</code></pre>

Sending a ping to IP address 2.2.2.2 and sourcing it from my own loopback interface proves that both routers know how to reach each other’s loopback interface. Since we don’t use the directly connected interfaces for the peering, we also have to increase the TTL:

<pre><code><strong>R1(config-router)#neighbor 2.2.2.2 ebgp-multihop 2
</strong></code></pre>

<pre><code><strong>R2(config-router)#neighbor 1.1.1.1 ebgp-multihop 2
</strong></code></pre>

The ebgp-multihop command changes the TTL to 2. Now take a look at a debug:

<pre><code><strong>R2#debug ip bgp 
</strong>BGP debugging is on for address family: IPv4 Unicast

BGPNSF state: 1.1.1.1 went from nsf_not_active to nsf_not_active
BGP: 1.1.1.1 went from Active to Idle
BGP: 1.1.1.1 went from Idle to Active
BGP: 1.1.1.1 open active delayed 31810ms (35000ms max, 28% jitter)
BGP: 1.1.1.1 open active, local address 192.168.12.2
BGP: 1.1.1.1 open failed: Connection refused by remote host, open active delayed 34480ms (35000ms max, 28% jitter)
</code></pre>

We can enable a debug to see the progress. You can clearly see that R2 is using IP address 192.168.12.2 and that R1 is refusing the connection. This is because we use the wrong source IP address. We have to tell BGP to use another IP address:

<pre><code><strong>R1(config-router)#neighbor 2.2.2.2 update-source loopback 0
</strong></code></pre>

<pre><code><strong>R2(config-router)#neighbor 1.1.1.1 update-source loopback 0
</strong></code></pre>

Use the update-source command to change the source IP address for the BGP updates. After making these changes, the problem should be fixed:

```
R1#
%BGP-5-ADJCHANGE: neighbor 2.2.2.2 Up
```

```
R2#
%BGP-5-ADJCHANGE: neighbor 1.1.1.1 Up
```

There goes! A working BGP neighbor adjacency.

**Lesson learned: BGP routers don’t have to establish a neighbor adjacency using the directly connected interfaces. Make sure the BGP routers can reach each other, that BGP packets are sourced from the correct interface and in case of EBGP don’t forget to use the multihop command.**

## BGP TCP Port Filtering

Let’s take a look at an IBGP issue:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/r1-r2-ibgp-as-1.png" alt=""><figcaption></figcaption></figure>

Two routers in the same AS and here’s the configuration:

<pre><code><strong>R1#show run | section bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.12.2 remote-as 1
 no auto-summary
</code></pre>

<pre><code><strong>R2#show run | section bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.12.1 remote-as 1
 no auto-summary
</code></pre>

Plain and simple. The routers use the directly connected IP addresses for the BGP neighbor adjacency. Let’see if we have neighbors:

<pre><code><strong>R1#show ip bgp summary 
</strong>BGP router identifier 192.168.12.1, local AS number 1
BGP table version is 1, main routing table version 1

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.2    4     1      46      46        0    0    0 00:05:24 Active 
</code></pre>

<pre><code><strong>R2#show ip bgp summary 
</strong>BGP router identifier 192.168.12.2, local AS number 1
BGP table version is 1, main routing table version 1

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.1    4     1      46      46        0    0    0 00:05:30 Active
</code></pre>

Too bad…we are not becoming neighbors. What could possibly be wrong? We are using the directly connected interfaces so there’s not that much that could go wrong except for L2/L2 issues. Let’s try a simple ping:

<pre><code><strong>R1#ping 192.168.12.2
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.12.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/4/4 ms
</code></pre>

Sending a ping from one router to the other proves that L2 and L3 are working fine. What about L4? We could have issues with the transport layer. Let’s give it a try:

<pre><code><strong>R1#telnet 192.168.12.2 179
</strong>Trying 192.168.12.2, 179 ... 
% Destination unreachable; gateway or host down
</code></pre>

<pre><code><strong>R2#telnet 192.168.12.1 179
</strong>Trying 192.168.12.1, 179 ...
</code></pre>

I’m unable to connect to TCP port 179 from both routers. This should ring a bell, maybe something is blocking BGP ?

<pre><code><strong>R1#show ip interface fastEthernet 0/0 | include access
</strong>  Outgoing access list is not set
  Inbound  access list is not set
</code></pre>

<pre><code><strong>R2#show ip interface fastEthernet 0/0 | include access
</strong>  Outgoing access list is not set
  Inbound  access list is 100
</code></pre>

Dang! It’s the security team…let’s take a closer look:

<pre><code><strong>R2#show ip access-lists 
</strong>Extended IP access list 100
    10 deny tcp any eq bgp any (293 matches)
    15 deny tcp any any eq bgp (153 matches)
    20 permit ip any any (109 matches)
</code></pre>

Someone decided it was a good idea to “secure” BGP and block it with an access-list. Let’s get rid of it:

<pre><code><strong>R2(config)#interface fastEthernet 0/0
</strong><strong>R2(config-if)#no ip access-group 100 in
</strong></code></pre>

Once I do this, everything is working again:

```
R1# %BGP-5-ADJCHANGE: neighbor 192.168.12.2 Up
```

```
R2# %BGP-5-ADJCHANGE: neighbor 192.168.12.1 Up
```

That’s what we are looking for!\
\
**Lesson learned: Don’t block BGP TCP port 179.**

## IBGP Update Source

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/r1-r2-ibgp-as-1-loopbacks.png" alt=""><figcaption></figcaption></figure>

Next IBGP issue. This one is similar to the EBGP situation I showed you before…we are using the loopback interfaces to establish the BGP neighbor adjacency, here are the configurations:

<pre><code><strong>R1#show run | section router bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 neighbor 2.2.2.2 remote-as 1
  no auto-summary
</code></pre>

<pre><code><strong>R2#show run | section router bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 neighbor 1.1.1.1 remote-as 1
  no auto-summary
</code></pre>

Nothing special, IBGP and we are using the loopback interfaces for the neighbor adjacency. There are no neighbors though:

<pre><code><strong>R1#show ip bgp summary | begin Neighbor
</strong>Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2.2.2.2         4     1       0       0        0    0    0 never    Active
</code></pre>

<pre><code><strong>R2#show ip bgp summary | begin Neighbor
</strong>Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4     1       0       0        0    0    0 never    Active
</code></pre>

No luck here…no neighbors.

Let’s first check if the routers can reach each other’s loopback interfaces:

<pre><code><strong>R1#show ip route
</strong>
C    192.168.12.0/24 is directly connected, FastEthernet0/0
     1.0.0.0/24 is subnetted, 1 subnets
C       1.1.1.0 is directly connected, Loopback0
</code></pre>

<pre><code><strong>R2#show ip route
</strong>
C    192.168.12.0/24 is directly connected, FastEthernet0/0
     2.0.0.0/24 is subnetted, 1 subnets
C       2.2.2.0 is directly connected, Loopback0
</code></pre>

A quick look at the routing table shows us that this is not the case. We could fix this with a static route or an IGP. Normally we use an IGP for IBGP to advertise the loopback interfaces, let’s use OSPF:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 1.1.1.0 0.0.0.255 area 0
</strong><strong>R1(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong><strong>R2(config-router)#network 2.2.2.0 0.0.0.255 area 0
</strong></code></pre>

Smashing in the correct OSPF commands should do the job! Let’s try a quick ping:

<pre><code><strong>R1#ping 2.2.2.2 source loopback 0
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2.2.2.2, timeout is 2 seconds:
Packet sent with a source address of 1.1.1.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/4/4 ms
</code></pre>

Reachability is no issue anymore. So do we have neighbors now?

<pre><code><strong>R1#show ip bgp summary | begin Neighbor
</strong>Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2.2.2.2         4     1       0       0        0    0    0 never    Active
</code></pre>

<pre><code><strong>R2#show ip bgp summary | begin Neighbor
</strong>Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4     1       0       0        0    0    0 never    Active
</code></pre>

Still no BGP neighbor adjacency though…let’s try a debug:

<pre><code><strong>R1#debug ip bgp 
</strong>BGP debugging is on for address family: IPv4 Unicast
BGP: 2.2.2.2 open active, local address 192.168.12.1
BGP: 2.2.2.2 open failed: Connection refused by remote host, open active delayed 32957ms (35000ms max, 28% jitter)
</code></pre>

<pre><code><strong>R2#debug ip bgp 
</strong>BGP debugging is on for address family: IPv4 Unicast
BGP: 1.1.1.1 open active, local address 192.168.12.2
BGP: 1.1.1.1 open failed: Connection refused by remote host, open active delayed 32957ms (35000ms max, 28% jitter)
</code></pre>

A debug shows up that the connection is refused and it also shows us the local IP address that is used for BGP. Seems someone forgot to add the update-source command so let’s fix it!

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 2.2.2.2 update-source loopback 0
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 1
</strong><strong>R2(config-router)#neighbor 1.1.1.1 update-source loopback 0
</strong></code></pre>

Just like EBGP we have to set the correct source for our BGP packets. After a few seconds you’ll see this:

```
R1# BGP-5-ADJCHANGE: neighbor 2.2.2.2 Up
```

```
R2# BGP-5-ADJCHANGE: neighbor 1.1.1.1 Up
```

Problem solved! The only difference here is that for iBGP peerings, we don’t need to change the TTL, whereas for eBGP we must use the `ebgp-multihop` command.

**Lesson learned: Its common practice to configure IBGP between loopback interfaces. Make sure these loopbacks are reachable and that the BGP updates are sourced from the loopback interface.**

These are the most common issues why BGP doesn’t form a neighbor adjacency. The routers are now up and running so we can continue to [troubleshoot (missing) route advertisements](https://networklessons.com/cisco/ccnp-encor-350-401/troubleshooting-bgp-route-advertisement).

If you have any questions about this lesson, feel free to leave a comment!
