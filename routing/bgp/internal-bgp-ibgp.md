# Internal BGP (iBGP)

In this lesson, we’ll take a look at IBGP (Internal BGP). Students who are new to BGP often wonder why we have “external” and “internal” BGP. I’m not going to show you just a couple of quick commands but we’ll take a close look at IBGP and its configuration.

Let’s start with an example topology and I’ll explain a couple of things:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/09/5-routers-3-AS1-AS2-AS3.png" alt=""><figcaption></figcaption></figure>

Above you see 3 autonomous systems and 5 routers. When AS1 wants to reach AS3 we have to cross AS2, this makes AS2 our transit AS. This is a typical scenario where AS1 and AS3 are customers and AS2 is the ISP.

In our scenario, AS1 has a loopback interface with network 1.1.1.0 /24 and AS3 wants to reach this network. This means we’ll have to advertise this network through BGP. Here’s what it looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/07/bgp-synchronization-example-topology-ibgp-ebgp.png" alt=""><figcaption></figcaption></figure>

&#x20;

So what is going on here? Let me explain it step-by-step:

1. We need EBGP between AS1 and AS2 because these are two different autonomous systems. This allows us to advertise a prefix on R1 in BGP so that AS2 can learn it.
2. We also need EBGP between AS2 and AS3 so that R5 can learn prefixes through BGP.
3. We need to get the prefix that R2 learned from R1 somehow to R5. We do this by configuring IBGP between R2 and R4, this allows R4 to advertise it to R5.

So that’s the first reason why we need IBGP…so you can **advertise a prefix from one autonomous system to another**. You might have a few questions after reading this:

1. Why don’t we use OSPF (or EIGRP) on AS2 instead and redistribute the prefix on R2 from BGP into OSPF and on R4 from OSPF back into BGP?
2. Doesn’t IBGP have to be directly connected?
3. How are R2 and R4 able to reach each other through IBGP if we don’t have any routing protocol within AS2?
4. What about R3? do we need IBGP?

These are some of the questions I get all the time from students who are learning BGP, here are the answers:

1. Technically this is possible…we can run OSPF (or EIGRP) within AS2 and use redistribution between BGP and OSPF. In my example R1 will only have a single prefix so it’s no problem but what if R1 had a full internet routing table? (over 500.000 prefixes since 2014). IGPs like OSPF or EIGRP are not able to handle that many prefixes so you’ll need BGP for this.
2. IBGP does not have to be directly connected, this might be a little confusing when you only know about OSPF or EIGRP since they always form adjacencies on directly connected links.
3. They are not! This is why we **need an IGP within the AS**. Since R2 and R4 are not directly connected we’ll configure an IGP so that they can reach each other.
4. I’ll give you the answer to this question in a bit…I want to show you what will go wrong if we don’t configure R3 🙂

Enough reading for now, let’s get our hands dirty with some configuration. We’ll start with BGP between R1/R2, R2/R4 and R4/5 like I just described.

## Configuration

First we’ll configure R1 and R2. I am also advertising a prefix (on a loopback interface) in BGP:

<pre><code><strong>R1(config)#interface loopback 0
</strong><strong>R1(config-if)#ip address 1.1.1.1 255.255.255.0
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong><strong>R1(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

<pre><code><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong></code></pre>

That’s easy enough, just a few commands. Our next step will be to configure IBGP between R2 and R4…what IP addresses are we going to use for this? Let’s look at our options:

<pre><code><strong>R2#show ip interface brief
</strong>Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.12.2    YES NVRAM  up                    up
FastEthernet1/0            192.168.23.2    YES NVRAM  up                    up
</code></pre>

<pre><code><strong>R4#show ip interface brief
</strong>Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.34.4    YES NVRAM  up                    up
FastEthernet1/0            192.168.45.4    YES NVRAM  up                    up
</code></pre>

I can use any of these IP addresses but we need connectivity. That’s why we need an IGP like we talked about earlier. So which IP addresses will we select? In this particular scenario it really doesn’t matter since there is only 1 path between R2 and R4. What if we had multiple paths between R2 and R4?

When there are **multiple paths it’s better to use a loopback interface** with an IP address and to advertise that into your IGP. We will use the loopback interface as the source for our BGP session. Why?

A physical interface can go down which means the IP address on the interface is no longer reachable. A **loopback interface will never go down** unless the router crashes or when you “shut” it. This is why it’s best practice to use loopback interfaces when configuring IBGP.

I’ll add a loopback interface on R2 and R4 and use these for IBGP, first we’ll have to configure an IGP (I’ll use OSPF) to advertise them:

<pre><code><strong>R2(config)#interface loopback 0
</strong><strong>R2(config-if)#ip address 2.2.2.2 255.255.255.0
</strong></code></pre>

<pre><code><strong>R4(config)#interface loopback 0
</strong><strong>R4(config-if)#ip address 4.4.4.4 255.255.255.0
</strong></code></pre>

That takes care of the loopback interfaces, now we can enable OSPF:

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 192.168.23.0 0.0.0.255 area 0
</strong><strong>R2(config-router)#network 2.2.2.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#network 192.168.23.0 0.0.0.255 area 0
</strong><strong>R3(config-router)#network 192.168.34.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R4(config)#router ospf 1
</strong><strong>R4(config-router)#network 192.168.34.0 0.0.0.255 area 0
</strong><strong>R4(config-router)#network 4.4.4.0 0.0.0.255 area 0
</strong></code></pre>

Excellent, R2 and R4 will now be able to reach each others loopback interfaces. It’s not a bad idea to test this though:

<pre><code><strong>R2#ping 4.4.4.4 source 2.2.2.2
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 4.4.4.4, timeout is 2 seconds:
Packet sent with a source address of 2.2.2.2
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 48/52/60 ms
</code></pre>

Alright we are now prepared for IBGP between R2 and R4. Here’s what it looks like:

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 4.4.4.4 remote-as 2
</strong><strong>R2(config-router)#neighbor 4.4.4.4 update-source loopback 0
</strong></code></pre>

<pre><code><strong>R4(config)#router bgp 2
</strong><strong>R4(config-router)#neighbor 2.2.2.2 remote-as 2
</strong><strong>R4(config-router)#neighbor 2.2.2.2 update-source loopback 0
</strong></code></pre>

This takes care of our IBGP session. Note that we have to use the **update-source** command to specify that we will use the loopback interfaces as the source for the IBGP session.

Last but not least, let’s configure EBGP between R4 and R5:

<pre><code><strong>R4(config)#router bgp 2
</strong><strong>R4(config-router)#neighbor 192.168.45.5 remote-as 3
</strong></code></pre>

<pre><code><strong>R5(config)#router bgp 3
</strong><strong>R5(config-router)#neighbor 192.168.45.4 remote-as 2
</strong></code></pre>

Great, that takes care of that. Whenever you configure BGP you will see a message on the console that shows you that the neighbor adjacency has been established. You can also check it with the **show ip bgp summary** command.

## Verification

If everything went OK, all routers should have learned about the 1.1.1.0 /24 prefix that I advertised on R1. Let’s see if that is true:

First we’ll check R1:

<pre><code><strong>R1#show ip bgp
</strong>BGP table version is 2, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       0.0.0.0                  0         32768 i
</code></pre>

You can see that it is in the BGP table. This means that I succesfully used the network command to advertise into BGP. The next hop is 0.0.0.0 since it originated on this router. If you don’t see anything here then normally there are two reasons for this:

* You can’t advertise something in BGP that is not in your routing table, make sure the interface is up/up.
* You typed an incorrect subnet mask when you used the network command (has to be exact match!).

Let’s see what R2 thinks about this:

<pre><code><strong>R2#show ip bgp
</strong>BGP table version is 2, local router ID is 2.2.2.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.12.1             0             0 1 i
</code></pre>

That’s looking good too. R2 knows about our prefix, you can see that the next hop is the IP address of R1. If you take a closer look you can see the **> symbol** in front of the prefix, this means that the router selected this entry as the best one and that it **installed it in the routing table**. Let’s check R4, it should receive this information from R2:

<pre><code><strong>R4#show ip bgp
</strong>BGP table version is 1, local router ID is 4.4.4.4
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
* i1.1.1.0/24       192.168.12.1             0    100      0 1 i
</code></pre>

R4 learned about the prefix but there’s something going on here…there is no > symbol before the prefix so R4 didn’t install this in the routing table. Can you tell why this is happening? Take a close look at the next hop…I’ll give you the answer in a sec, let’s check R5 first:

<pre><code><strong>R5#show ip bgp
</strong></code></pre>

There’s nothing in R5…that’s because R4 is having some issues, look closely:

```
   Network          Next Hop            Metric LocPrf Weight Path
* i1.1.1.0/24       192.168.12.1             0    100      0 1 i
```

Does R4 have any idea how to reach the next hop? **BGP doesn’t change the next hop IP address by default** so this can cause some issues. Let’s verify if R4 knows how to reach the next hop:

<pre><code><strong>R4#show ip route 192.168.12.1
</strong>% Network not in table
</code></pre>

No next hop, so we can’t install the prefix from BGP into the routing table…how are we going to fix this? As always there are multiple options:

* Advertise network 192.168.12.0 /24 in a routing protocol (IGP or BGP).
* Change the next hop IP address with the **next-hop-self command**.

I’ll change the next hop IP address since it’s a good practice, here’s how it works:

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 4.4.4.4 next-hop-self
</strong></code></pre>

<pre><code><strong>R4(config)#router bgp 2
</strong><strong>R4(config-router)#neighbor 2.2.2.2 next-hop-self
</strong></code></pre>

I’m doing this on both R2 and R4. For this scenario I don’t have to do it but if I would advertise something on R5 then R2 would have the same problem as R4. Take a look again R4 to see the changes:

<pre><code><strong>R4#show ip bgp
</strong>BGP table version is 2, local router ID is 4.4.4.4
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*>i1.1.1.0/24       2.2.2.2                  0    100      0 1 i
</code></pre>

Excellent…two important changes here. First of all you see the > symbol which means R4 was able to install this prefix in the routing table. Secondly, the next hop IP address has been changed to something R4 knows (the loopback interface of R2).

Since R4 is now able to install it in the routing table, it can advertise the prefix to R5:

<pre><code><strong>R5#show ip bgp
</strong>BGP table version is 2, local router ID is 5.5.5.5
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.45.4                           0 2 1 i
</code></pre>

R5 has learned about the prefix…so far so good, you can see that it’s in the routing table:

<pre><code><strong>R5#show ip route bgp
</strong>     1.0.0.0/24 is subnetted, 1 subnets
B       1.1.1.0 [20/0] via 192.168.45.4, 00:02:08
</code></pre>

That’s looking good. So are we done? Is there connectivity? Let’s find out:

<pre><code><strong>R5#ping 1.1.1.1
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
</code></pre>

Uh-oh…something went wrong. This is often a very frustrating moment for many BGP students, they see something in the routing table but it doesn’t work. What is going on here?

Let’s do a quick trace from R5 to see how far we can get to R1:

<pre><code><strong>R5#traceroute 1.1.1.1
</strong>
Type escape sequence to abort.
Tracing the route to 1.1.1.1

  1 192.168.45.4 52 msec 16 msec 32 msec
  2  *  *  *
  3  *  *  *
</code></pre>

So our IP packet reaches R4 but after that it went somewhere into oblivion. R4 is not the problem so we’ll have to check the next device in the path towards R1, that’s R3.

R3 is an interesting router since it doesn’t run BGP, only OSPF. Let’s check R3:

<pre><code><strong>R3#show ip route 1.1.1.0
</strong>% Network not in table
</code></pre>

There’s our problem, R3 receives an IP packet with destination 1.1.1.1 but has no clue where to send it so it will be dropped. How do we fix this?

Once again, you could redistribute BGP into OSPF but that’s a bad idea…1 prefix could work but an entire internet routing table…not gonna happen!

This is why you need **IBGP on all your routers in your transit AS**. We need to configure IBGP on R3 so it learns about our 1.1.1.0 /24 prefix and it will know how to reach the destination.

Just like R2 and R4, I’ll use a loopback interface on R3 as the source of our IBGP session.

I will configure IBGP between R2/R3 and R3/R4.  Let’s create a loopback, advertise it in OSPF and configure BGP:

<pre><code><strong>R3(config)#interface loopback 0
</strong><strong>R3(config-if)#ip address 3.3.3.3 255.255.255.0
</strong></code></pre>

<pre><code><strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#network 3.3.3.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R3(config)#router bgp 2
</strong><strong>R3(config-router)#neighbor 2.2.2.2 remote-as 2
</strong><strong>R3(config-router)#neighbor 2.2.2.2 update-source loopback 0
</strong><strong>R3(config-router)#neighbor 4.4.4.4 remote-as 2
</strong><strong>R3(config-router)#neighbor 4.4.4.4 update-source loopback 0
</strong></code></pre>

That takes care of R3, now we’ll configure R2 and R4 to peer with R3:

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 3.3.3.3 remote-as 2
</strong><strong>R2(config-router)#neighbor 3.3.3.3 update-source loopback 0
</strong><strong>R2(config-router)#neighbor 3.3.3.3 next-hop-self
</strong></code></pre>

<pre><code><strong>R4(config)#router bgp 2
</strong><strong>R4(config-router)#neighbor 3.3.3.3 remote-as 2
</strong><strong>R4(config-router)#neighbor 3.3.3.3 update-source loopback 0
</strong><strong>R4(config-router)#neighbor 3.3.3.3 next-hop-self
</strong></code></pre>

This will establish IBGP between R2/R3 and R3/R4. Take a look at the BGP table of R3:

<pre><code><strong>R3#show ip bgp
</strong>BGP table version is 2, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*>i1.1.1.0/24       2.2.2.2                  0    100      0 1 i
</code></pre>

Very nice…R3 now knows how to reach the 1.1.1.0 /24 network so it’s no longer the problem. Can R5 finally reach R1? Let’s find out:

<pre><code><strong>R5#ping 1.1.1.1
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
</code></pre>

It still doesn’t work, this is where the frustration turns into a BGP hate rage (just kidding hehe). I’ll show you what the problem is here…

It’s a good idea to check some of the routers that are closer to R1, see if they are able to ping 1.1.1.1. Let’s start with R2:

<pre><code><strong>R2#ping 1.1.1.1
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/20/24 ms
</code></pre>

No problem for R2, what about R3?

<pre><code><strong>R3#ping 1.1.1.1
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
</code></pre>

R3 is unable to reach R1…interesting! Previously we checked the BGP and routing table of R3 and it has all information required to reach R1. What could go wrong here? The problem is not R3 but it’s R1..take a look here:

<pre><code><strong>R1#show ip route
</strong>Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

C    192.168.12.0/24 is directly connected, FastEthernet0/0
     1.0.0.0/24 is subnetted, 1 subnets
C       1.1.1.0 is directly connected, Loopback0
</code></pre>

This is all that R1 has in its routing table. What happens is that R1 receives an IP packet from R3 that looks like this:

* Source IP address: 192.168.23.3
* Destination IP address: 1.1.1.1

When R1 wants to reply to 192.168.23.3 it has no clue where to send it…it’s not in its routing table! If you want you can verify this with a debug:

<pre><code><strong>R1#debug ip packet
</strong>IP packet debugging is on
</code></pre>

This will show us what happens when R1 receives the IP packet. Don’t do this on a production router as it will produce way too much debug information:

```
R1#
IP: s=1.1.1.1 (local), d=192.168.23.3, len 100, unroutable
```

R1 says it’s unroutable, the destination is unknown. To fix this problem we have to advertise some additional networks. I don’t really care about R3 being able to reach R1 but I do want R5 to reach R1.

What we’ll do is advertise the 192.168.45.0 /24 prefix into BGP, we can do this on R4 or R5:

<pre><code><strong>R5(config)#router bgp 3
</strong><strong>R5(config-router)#network 192.168.45.0 mask 255.255.255.0
</strong></code></pre>

Let’s see if R1 learns this prefix:

<pre><code><strong>R1#show ip bgp
</strong>BGP table version is 3, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       0.0.0.0                  0         32768 i
*> 192.168.45.0     192.168.12.2                           0 2 3 i
</code></pre>

It’s in the BGP table and also in the routing table:

<pre><code><strong>R1#show ip route bgp
</strong>B    192.168.45.0/24 [20/0] via 192.168.12.2, 00:00:50
</code></pre>

Let’s try a ping:

<pre><code><strong>R5#ping 1.1.1.1
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 84/104/120 ms
</code></pre>

Finally! it’s working! If you also want to ping R1 from any of the other routers then you need to make sure R1 knows where to send the return traffic.

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
interface Loopback0
 ip address 1.1.1.1 255.255.255.0
!
interface FastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
router bgp 1
 bgp log-neighbor-changes
 network 1.1.1.0 mask 255.255.255.0
 neighbor 192.168.12.2 remote-as 2
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
interface Loopback0
 ip address 2.2.2.2 255.255.255.0
!
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
interface FastEthernet0/1
 ip address 192.168.23.2 255.255.255.0
!
router ospf 1
 network 2.2.2.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
!
router bgp 2
 bgp log-neighbor-changes
 neighbor 3.3.3.3 remote-as 2
 neighbor 3.3.3.3 update-source Loopback0
 neighbor 3.3.3.3 next-hop-self
 neighbor 4.4.4.4 remote-as 2
 neighbor 4.4.4.4 update-source Loopback0
 neighbor 4.4.4.4 next-hop-self
 neighbor 192.168.12.1 remote-as 1
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
interface Loopback0
 ip address 3.3.3.3 255.255.255.0
!
interface FastEthernet0/0
 ip address 192.168.23.3 255.255.255.0
!
interface FastEthernet0/1
 ip address 192.168.34.3 255.255.255.0
!
router ospf 1
 network 3.3.3.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
 network 192.168.34.0 0.0.0.255 area 0
!
router bgp 2
 bgp log-neighbor-changes
 neighbor 2.2.2.2 remote-as 2
 neighbor 2.2.2.2 update-source Loopback0
 neighbor 4.4.4.4 remote-as 2
 neighbor 4.4.4.4 update-source Loopback0
!
end
```
{% endtab %}

{% tab title="R4" %}
```
hostname R4
!
ip cef
!
interface Loopback0
 ip address 4.4.4.4 255.255.255.0
!
interface FastEthernet0/0
 ip address 192.168.34.4 255.255.255.0
!
interface FastEthernet0/1
 ip address 192.168.45.4 255.255.255.0
!
router ospf 1
 network 4.4.4.0 0.0.0.255 area 0
 network 192.168.34.0 0.0.0.255 area 0
!
router bgp 2
 bgp log-neighbor-changes
 neighbor 2.2.2.2 remote-as 2
 neighbor 2.2.2.2 update-source Loopback0
 neighbor 2.2.2.2 next-hop-self
 neighbor 3.3.3.3 remote-as 2
 neighbor 3.3.3.3 update-source Loopback0
 neighbor 3.3.3.3 next-hop-self
 neighbor 192.168.45.5 remote-as 3
!
end
```
{% endtab %}

{% tab title="R5" %}
```
hostname R5
!
ip cef
!
interface FastEthernet0/0
 ip address 192.168.45.5 255.255.255.0
!
router bgp 3
 bgp log-neighbor-changes
 neighbor 192.168.45.4 remote-as 2
 network 192.168.45.0 mask 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

Are we done now? Almost…there’s one more thing I want to teach you about the IBGP neighbor adjacencies…

## IBGP Neighbor Adjacencies

Right now our routers within AS2 are configured like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/09/R2-R3-R4-full-mesh-IBGP.png" alt=""><figcaption></figcaption></figure>

This is called **full-mesh IBGP**. All routers within AS 2 are neighbors with each other. Do we really need the IBGP peering between R2 and R4? Let’s find out what happens when I remove it…

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#no neighbor 4.4.4.4
</strong></code></pre>

<pre><code><strong>R4(config)#router bgp 2
</strong><strong>R4(config-router)#no neighbor 2.2.2.2
</strong></code></pre>

Just to visualize it, our picture now looks like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/09/R2-R3-R4-partial-mesh-IBGP.png" alt=""><figcaption></figcaption></figure>

Let’s check out the BGP table of R3 to see what it has:

<pre><code><strong>R3#show i²bgp
</strong>BGP table version is 3, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*>i1.1.1.0/24       2.2.2.2                  0    100      0 1 i
*>i192.168.45.0     4.4.4.4                  0    100      0 3 i
</code></pre>

R3 learned about 1.1.1.0 /24 from R2 and 192.168.45.0 /24 from R4. This is good, these are prefixes that we advertised before. Now look at R2:

<pre><code><strong>R2#show ip bgp
</strong>BGP table version is 4, local router ID is 2.2.2.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.12.1             0             0 1 i
</code></pre>

R2 only knows about 1.1.1.0 /24, it didn’t learn about 192.168.45.0 /24 from R3. What about R4?

<pre><code><strong>R4#show ip bgp
</strong>BGP table version is 5, local router ID is 4.4.4.4
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
r> 192.168.45.0     192.168.45.5             0             0 3 i
</code></pre>

R4 only learned about 192.168.45.0 /24 from R5, we don’t see 1.1.1.0 /24 here.

The problem here is that **IBGP does not advertise prefixes from one IBGP neighbor to another IBGP neighbor**. This is called **BGP split horizon**.

There is a good reason why IBGP works like this…

Between different ASes, BGP uses the AS\_PATH attribute to avoid routing loops. A prefix will not be accepted by a BGP router if it sees its own AS number in it…plain and simple. However, within the autonomous system the AS number does not change so we can’t use this loop prevention mechanism.

Without BGP split horizon, a route could be advertised like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/09/ibgp-no-split-horizon.png" alt=""><figcaption></figcaption></figure>

R1 could receive an update about a prefix that it originated itself…not a good idea. With BGP split horizon this can’t occur:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/09/ibgp-split-horizon.png" alt=""><figcaption></figcaption></figure>

R2 will never forward the IBGP prefixes that it learns from R1 towards R3. This means that **all your IBGP routers have to become neighbors with all other IBGP routers** (full-mesh!). If you have a lot of IBGP routers then this can be a lot of work, the number of required adjacencies is:

$$X × (X-1)/2$$

So with 10 IBGP routers you will need to configure 45 IBGP neighbor adjacencies. There are two techniques to reduce this number:

* BGP Route Reflectors
* BGP Confederations

I will explain both in other lessons in the future! This is the end of the IBGP explanation, I hope you enjoyed it and learned a thing or two. If you have any questions feel free to leave a comment!
