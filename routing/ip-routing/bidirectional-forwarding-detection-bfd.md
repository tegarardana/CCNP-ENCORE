# Bidirectional Forwarding Detection (BFD)

BFD (Bidirectional Forwarding Detection) is a super fast protocol that is able to detect link failures within milliseconds or even microseconds.. All (routing) protocols have some sort of mechanism to detect link failures. OSPF uses hello packets and a dead interval, EIGRP uses hello packets and a holddown timer etc.

Networks that use real-time traffic like VoIP require fast convergence times. Routing protocols like OSPF or EIGRP are able to quickly select another path once they lose a neighbor but it takes a while for them to realize that something is wrong.

We can tune timers for fast convergence, for example OSPF can be configured to use a dead interval of only one second. The problem however is that all of these protocols were never really designed for sub-second failover. Hello packets and such are processed by the control plane so there is quite some overhead. BFD was designed to be fast, its packets can be processed by some interface modules or line cards so there isn’t much overhead.

BFD runs independent from any other (routing) protocols. Once it’s up and running, you can configure protocols like OSPF, EIGRP, BGP, HSRP, MPLS LDP etc. to use BFD for link failure detection instead of their own mechanisms. When the link fails, BFD will inform the protocol. Here’s how you can visualize this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/ospf-bfd-r1-r2.png" alt=""><figcaption></figcaption></figure>

R1 and R2 are configured to use BFD and will send control packets to each other. OSPF remains the same, it’s sending its OSPF packets. Once the link fails, this will happen:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/bfd-informs-ospf-link-failure.png" alt=""><figcaption></figcaption></figure>

&#x20;When BFD doesn’t receive its control packets anymore it realizes we have a link failure and it will report this to OSPF. OSPF will then tear down the neighbor adjacency.

There are two operating modes to BFD, **asynchronous mode** and **demand mode**. The asynchronous mode is similar to the hello and holddown timers, BFD will keep sending hello packets (called BFD control packets) and when you don’t receive some of them, the session is teared down.

The demand mode is different, once BFD has found a neighbor it won’t continuously send control packets but only uses a polling mechanism. Another method has to be used to check reachability, for example it could check the receive and transmit statistics of the interface. Right now Cisco (or any other vendor I know of) doesn’t support BFD demand mode.

Both modes also support something called **echo mode**. When a device sends BFD echo packets then the receiver will return them without processing them. When the sender doesn’t get the echo packets back, it knows something is wrong and will tear down the session.

Anyway enough talk about BFD for now, let’s take a look at this in action!

## Configuration

To see why BFD is great, we will look at a scenario with and without BFD. I’ll use OSPF but many other (routing) protocols can be used.

### OSPF without BFD

This is the topology that we will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/bfd-r1-sw1-r2.png" alt=""><figcaption></figcaption></figure>

Above we have two routers that are connected to a switch and running OSPF. Here’s the configuration:

<pre><code>R1 &#x26; R2
<strong>(config)#router ospf 1
</strong><strong>(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong></code></pre>

Nothing special, just regular OSPF. Suddenly the link fails:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/bfd-link-failure-r1-sw1.png" alt=""><figcaption></figcaption></figure>

Here’s what will happen:

```
R1#
Jul 30 11:54:46.011: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to down
Jul 30 11:54:46.011: %OSPF-5-ADJCHG: Process 1, Nbr 192.168.12.2 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: Interface down or detached
```

R1 will detect this link failure right away since it’s directly connected to SW1. It will immediately drop the neighbor adjacency. What about R2?

```
R2#
Jul 30 11:55:14.667: %OSPF-5-ADJCHG: Process 1, Nbr 192.168.12.1 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: Dead timer expired
```

R2 stops receiving OSPF hello packets from R1 so once the dead interval expires, it decides that R1 is unreachable and it will drop the neighbor adjacency. This took about 28 seconds.

Even if you [tune the OSPF timers,](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-hello-and-dead-interval) it will still take about one second. Let’s see how BFD performs…

### OSPF with BFD

Let’s enable BFD on our two routers running OSPF. Here’s the topology again:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/r1-sw1-r2.png" alt=""><figcaption></figcaption></figure>

Let’s take a close look at the BFD command:

<pre><code><strong>R1(config)#interface FastEthernet 0/0
</strong><strong>R1(config-if)#bfd ?
</strong>  echo      Use echo adjunct as bfd detection mechanism
  interval  Transmit interval between BFD packets
</code></pre>

Above we can specify that we want to use echo packets and we can set an interval. BFD uses echo packets by default so we don’t have to change this. Let’s look at the interval:

<pre><code><strong>R1(config)#interface FastEthernet 0/0
</strong><strong>R1(config-if)#bfd interval ?
</strong>  &#x3C;50-999>  Milliseconds
</code></pre>

The BFD interval is to specify how often we will send BFD packets, this is similar to the hello packet that protocols like OSPF, EIGRP, HSRP, etc. use. Let’s use the lowest value:

<pre><code><strong>R1(config-if)#bfd interval 50 ?
</strong>  min_rx  Minimum receive interval capability
</code></pre>

The second value to configure is the minimum receive interval. This is how often we expect to receive a BFD packet from our neighbor. We will use the same value here:

<pre><code><strong>R1(config-if)#bfd interval 50 min_rx 50 ?
</strong>  multiplier  Multiplier value used to compute holddown
</code></pre>

The last value to configure is for the holddown. This is similar to the dead interval in OSPF or the holddown time that other protocols use:

<pre><code><strong>R1(config-if)#bfd interval 50 min_rx 50 multiplier 3
</strong></code></pre>

We’ll set it to 3. Let’s configure the same on R2:

<pre><code><strong>R2(config)#interface FastEthernet 0/0
</strong><strong>R2(config-if)#bfd interval 50 min_rx 50 multiplier 3
</strong></code></pre>

BFD is now up and running but we still have to configure our protocols to actually use it. Here’s how to do this for OSPF:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#bfd all-interfaces 
</strong></code></pre>

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#bfd all-interfaces
</strong></code></pre>

OSPF will now tear down the neighbor adjacency as soon as BFD is telling it that there is a link failure. This is everything we have to do, let’s verify our work.

## Verification

Let’s start with some of the BFD commands:

<pre><code><strong>R1#show bfd neighbors 
</strong>
NeighAddr                         LD/RD    RH/RS     State     Int
192.168.12.2                       1/1     Up        Up        Fa0/0
</code></pre>

Above you get a quick overview with all BFD neighbors. You can see that R2 is found on FastEthernet 0/0 and that it’s up. Let’s take a closer look:

<pre><code><strong>R1#show bfd neighbors details 
</strong>
NeighAddr                         LD/RD    RH/RS     State     Int
192.168.12.2                       1/1     Up        Up        Fa0/0
Session state is UP and using echo function with 50 ms interval.
OurAddr: 192.168.12.1   
Local Diag: 0, Demand mode: 0, Poll bit: 0
MinTxInt: 1000000, MinRxInt: 1000000, Multiplier: 3
Received MinRxInt: 1000000, Received Multiplier: 3
Holddown (hits): 0(0), Hello (hits): 1000(617)
Rx Count: 96, Rx Interval (ms) min/max/avg: 1/1000/868 last: 648 ms ago
Tx Count: 618, Tx Interval (ms) min/max/avg: 1/1000/872 last: 708 ms ago
Elapsed time watermarks: 0 0 (last: 0)
Registered protocols: OSPF
Uptime: 00:01:23
Last packet: Version: 1                  - Diagnostic: 0
             State bit: Up               - Demand bit: 0
             Poll bit: 0                 - Final bit: 0
             Multiplier: 3               - Length: 24
             My Discr.: 1                - Your Discr.: 1
             Min tx interval: 1000000    - Min rx interval: 1000000
             Min Echo interval: 50000
</code></pre>

Above you can see how often BFD has sent and received packets but also which protocols are using BFD. Now let’s actually create a link failure to see how useful BFD is:

<pre><code><strong>SW1(config)#interface FastEthernet 0/1
</strong><strong>SW1(config-if)#shutdown
</strong></code></pre>

Almost immediately you will see these messages on R1 and R2:

```
R1#
Jul 30 13:11:33.507: %OSPF-5-ADJCHG: Process 1, Nbr 192.168.12.2 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: BFD node down
Jul 30 13:11:34.339: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to down
```

```
R2#
Jul 30 13:11:29.055: %OSPF-5-ADJCHG: Process 1, Nbr 192.168.12.1 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: BFD node down
```

Within a second, BFD reports to OSPF that there is a link failure and the neighbor adjacency has been dropped. Now that’s pretty quick!

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip cef
!
interface FastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
 bfd interval 50 min_rx 50 multiplier 3
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 bfd all-interfaces
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
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
 bfd interval 50 min_rx 50 multiplier 3
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 bfd all-interfaces
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

You have now seen how BFD can be used to speed up convergence because it is able to quickly detect link failures. It’s easy to configure and much faster than the “old” method of detecting link failures that protocols like OSPF, EIGRP, BGP, HSRP, etc. use.

I hope you enjoyed this lesson, if you have any questions feel free to leave a comment!
