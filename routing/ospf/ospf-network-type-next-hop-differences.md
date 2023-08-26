# OSPF Network Type Next Hop Differences

OSPF will use different IP addresses for the next hop depending on the network type that you use. This can be confusing when you try configuring OSPF on top of a frame-relay network. In this short lesson, I want to show you the difference between the next-hop IP address and the OSPF network type that we use.

There are 5 OSPF network types:

* <mark style="color:blue;">Non-Broadcast</mark>
* <mark style="color:blue;">Broadcast</mark>
* <mark style="color:red;">Point-to-Multipoint</mark>
* <mark style="color:red;">Point-to-Multipoint Non-Broadcast</mark>
* <mark style="color:red;">Point-to-Point</mark>

I made non-broadcast and broadcast blue because these two network types have something in common. They both require a DR/BDR election and basically, you are telling OSPF that you have a multi-access network. In other words…every router can reach any other router. This can be challenging with frame-relay because this is not always the case if you have a partial mesh or hub and spoke configuration.

Point-to-multipoint, point-to-multipoint non-broadcast, and point-to-point are in red because they have one thing in common: We tell OSPF that we have a “bunch of point-to-point” links. As you will see in my demonstration, the “blue” network types have different next-hop behavior from the “red” network types.

Let’s take a look at a configuration so I can demonstrate it to you:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-frame-relay-1.png" alt=""><figcaption></figcaption></figure>

Above you see a frame-relay hub and spoke network. We have two PVCs, but there is only a single subnet, so this is a frame-relay point-to-multipoint network. You can also see that router Spoke1 has network 2.2.2.0 /24 behind it. We will advertise this network into OSPF and see what the next hop IP address is like. Let’s configure our routers:

<pre><code><strong>Hub(config)#interface serial 0/0
</strong><strong>Hub(config-if)#ip address 192.168.123.1 255.255.255.0
</strong><strong>Hub(config-if)#encapsulation frame-relay 
</strong><strong>Hub(config-if)#ip ospf network broadcast 
</strong><strong>Hub(config-if)#exit
</strong><strong>Hub(config)#router ospf 1
</strong><strong>Hub(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>Spoke1(config)#interface serial 0/0
</strong><strong>Spoke1(config-if)#ip address 192.168.123.2 255.255.255.0
</strong><strong>Spoke1(config-if)#encapsulation frame-relay 
</strong><strong>Spoke1(config-if)#ip ospf network broadcast 
</strong><strong>Spoke1(config-if)#ip ospf priority 0
</strong><strong>Spoke1(config-if)#exit
</strong><strong>Spoke1(config)#router ospf 1
</strong><strong>Spoke1(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>Spoke2(config)#interface serial 0/0
</strong><strong>Spoke2(config-if)#ip address 192.168.123.3 255.255.255.0
</strong><strong>Spoke2(config-if)#encapsulation frame-relay 
</strong><strong>Spoke2(config-if)#ip ospf network broadcast 
</strong><strong>Spoke2(config-if)#ip ospf priority 0
</strong><strong>Spoke2(config-if)#exit
</strong><strong>Spoke2(config)#router ospf 1
</strong><strong>Spoke2(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

Above you see my configuration for the hub and spoke routers. I configured frame-relay, used the broadcast network type, and made sure that the spoke routers won’t become the DR/BDR with the `priority 0` command. Now let’s advertise network 2.2.2.0 /24 in OSPF on router Spoke1:

<pre><code><strong>Spoke1(config)#interface loopback 0
</strong><strong>Spoke1(config-if)#ip address 2.2.2.2 255.255.255.0    
</strong><strong>Spoke1(config-if)#exit
</strong><strong>Spoke1(config)#router ospf 1
</strong><strong>Spoke1(config-router)#network 2.2.2.0 0.0.0.255 area 0
</strong></code></pre>

We will add a loopback interface to advertise the network in OSPF. Let’s check router Spoke2 to see what it looks like in the routing table:

<pre><code><strong>Spoke2#show ip route ospf 
</strong>     2.0.0.0/32 is subnetted, 1 subnets
O       2.2.2.2 [110/65] via 192.168.123.2, 00:00:38, Serial0/0
</code></pre>

Above, we see the entry in the routing table of router Spoke2. Take a close look at the next hop IP address. 192.168.123.2 is the IP address of router Spoke1. Since we are using the broadcast network type, OSPF thinks that each router can reach any other router. In reality, there is only a PVC between the Hub and Spoke1 and between the Hub and Spoke2. What happens when we try to reach this network? Let’s find out!

<pre><code><strong>Spoke2#ping 2.2.2.2
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2.2.2.2, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
</code></pre>

We can’t reach it…why? First, check if the next hop IP address is reachable:

<pre><code><strong>Spoke2#ping 192.168.123.2
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.123.2, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
</code></pre>

The next hop IP address is unreachable. Let me show you why:

<pre><code><strong>Spoke2#show frame-relay map 
</strong>Serial0/0 (up): ip 192.168.123.1 dlci 301(0x12D,0x48D0), dynamic,
              broadcast,, status defined, active
</code></pre>

There’s only a frame-relay map for the hub router, so the spoke routers have no clue how to reach each other. In order to fix this, we’ll have to create two additional frame-relay maps.

<pre><code><strong>Spoke1(config)#interface serial 0/0
</strong><strong>Spoke1(config-if)#frame-relay map ip 192.168.123.3 201
</strong></code></pre>

<pre><code><strong>Spoke2(config)#interface serial 0/0
</strong><strong>Spoke2(config-if)#frame-relay map ip 192.168.123.2 301
</strong></code></pre>

This will help the spoke routers to reach each other.

<pre><code><strong>Spoke2#ping 2.2.2.2
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2.2.2.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/10/20 ms
</code></pre>

There we go…problem solved! Now let’s switch our OSPF network type to one of the “point” types to see the difference with the next hop IP address:

<pre><code><strong>Hub(config)#interface serial 0/0
</strong><strong>Hub(config-if)#ip ospf network point-to-multipoint
</strong></code></pre>

<pre><code><strong>Spoke1(config)#interface serial 0/0
</strong><strong>Spoke1(config-if)#ip ospf network point-to-multipoint
</strong></code></pre>

<pre><code><strong>Spoke2(config)#interface serial 0/0
</strong><strong>Spoke2(config-if)#ip ospf network point-to-multipoint
</strong></code></pre>

I will use the point-to-multipoint network type. Now let’s check the next hop IP address to see if there’s a difference:

<pre><code><strong>Spoke2#show ip route ospf | include 2.2.2.2
</strong>O       2.2.2.2 [110/129] via 192.168.123.1, 00:09:37, Serial0/0
</code></pre>

The next hop IP address is now the Hub router. To emphasize this, let me put the two next-hop IP addresses below each other.

Broadcast:

<pre><code><strong>Spoke2#show ip route ospf 
</strong>     2.0.0.0/32 is subnetted, 1 subnets
O       2.2.2.2 [110/65] via 192.168.123.2, 00:00:38, Serial0/0
</code></pre>

Point-to-Multipoint:

<pre><code><strong>Spoke2#show ip route ospf | include 2.2.2.2
</strong>O       2.2.2.2 [110/129] via 192.168.123.1, 00:09:37, Serial0/0
</code></pre>

So, in short…the “broadcast” and “non-broadcast” network types will use the next hop IP address of the router advertising the network, and you might have to create additional frame-relay maps to solve reachability issues. The “point” network types will use the next hop IP address of the router from which we received the information.

That’s all for now! I hope you enjoyed this lesson, if you have any questions just leave a comment.
