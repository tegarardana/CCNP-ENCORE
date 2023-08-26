# OSPFv3 IPv6 Troubleshooting Neighbor Adjacencies

In this lesson we’ll take a look at some OSPFv3 neighbor adjacency issues. Most of what you learned about OSPFv2 for IPv4 also applies to OSPFv3. Let’s take a look at the first issue!

## OSPFv3 Router ID

I will use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/ipv6-ospfv3-two-routers-area-0.png" alt=""><figcaption></figcaption></figure>

In the topology above we have 2 routers and there’s only a single area. For some reason the two routers are unable to become OSPF neighbors, up to us to find out why! Let’s check the interfaces first:

<pre><code><strong>R1#show ipv6 interface brief 
</strong>FastEthernet0/0            [up/up]
    FE80::CE00:1BFF:FE29:0
</code></pre>

<pre><code><strong>R2#show ipv6 interface brief 
</strong>FastEthernet0/0            [up/up]
    FE80::CE01:1BFF:FE29:0
</code></pre>

IPv6 routing protocols use the link-local addresses for neighbor adjacency and next-hops. We can see that both interfaces have a link-local IPv6 address and are active (up/up). Just in case, let’s ping the other side just to be sure that we have connectivity:

<pre><code><strong>R1#ping FE80::CE01:1BFF:FE29:0
</strong>Output Interface: FastEthernet0/0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to FE80::CE01:1BFF:FE29:0, timeout is 2 seconds:
Packet sent with a source address of FE80::CE00:1BFF:FE29:0
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/4/8 ms
</code></pre>

No issues there, let’s continue by checking OSPFv3:

<pre><code><strong>R1#show ipv6 protocols 
</strong>IPv6 Routing Protocol is "connected"
IPv6 Routing Protocol is "static"
IPv6 Routing Protocol is "ospf 1"
  Interfaces (Area 0):
    FastEthernet0/0 
</code></pre>

<pre><code><strong>R2#show ipv6 protocols 
</strong>IPv6 Routing Protocol is "connected"
IPv6 Routing Protocol is "static"
IPv6 Routing Protocol is "ospf 1"
  Interfaces (Area 0):
    FastEthernet0/0
</code></pre>

OSPFv3 is running on the FastEthernet0/0 interfaces of R1 and R2. No issues there, let’s check if we have neighbors or not:

<pre><code><strong>R1#show ipv6 ospf neighbor  
</strong>%OSPFv3: Router process 1 is INACTIVE, please configure a router-id
</code></pre>

<pre><code><strong>R2#show ipv6 ospf neighbor 
</strong>%OSPFv3: Router process 1 is INACTIVE, please configure a router-id
</code></pre>

This command reveals it all. We find out that the router-id has not been configured. OSPFv3 requires an I**Pv4 address-style router-ID** and we need to configure it ourselves. Let’s do that:

<pre><code><strong>R1(config-rtr)#router-id ?
</strong>  A.B.C.D  OSPF router-id in IP address format

<strong>R1(config-rtr)#router-id 1.1.1.1
</strong></code></pre>

<pre><code><strong>R2(config)#ipv6 router ospf 1
</strong><strong>R2(config-rtr)#router-id 2.2.2.2
</strong></code></pre>

The router-ID has to be an IPv4 address format. I have no idea why they decided to do it this way for OSPFv3 but my best guess is someone got a bit nostalgic. Anyway this fixes the issue:

```
R1# %OSPFv3-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet0/0 from LOADING to FULL, Loading Done
```

```
R2# %OSPFv3-5-ADJCHG: Process 1, Nbr 1.1.1.1 on FastEthernet0/0 from LOADING to FULL, Loading Done
```

After configuring the router-ID we almost immediately see a message that the OSPFv3 neighbor adjacency has been established. Let’s verify this:

<pre><code><strong>R1#show ipv6 ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
2.2.2.2           1   FULL/DR         00:00:33    4           FastEthernet0/0
</code></pre>

<pre><code><strong>R2#show ipv6 ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
1.1.1.1           1   FULL/BDR        00:00:31    4           FastEthernet0/0
</code></pre>

Problem solved!

**Lesson learned: Make sure you configure a router-ID for OSPFv3.**

## OSPFv3 Hello Packet Mismatch

Let’s look at another issue, same topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/ipv6-ospfv3-two-routers-area-0.png" alt=""><figcaption></figcaption></figure>

R1 and R2 are once again unable to form an OSPFv3 neighbor adjacency. Let’s check a couple of things:

<pre><code><strong>R1#show ipv6 interface brief 
</strong>FastEthernet0/0            [up/up]
    FE80::CE00:1BFF:FE29:0
</code></pre>

<pre><code><strong>R2#show ipv6 interface brief 
</strong>FastEthernet0/0            [up/up]
    FE80::CE01:1BFF:FE29:0
</code></pre>

The interfaces and IPv6 addresses are fine. Let’s do a quick ping:

<pre><code><strong>R1#ping FE80::CE01:1BFF:FE29:0
</strong>Output Interface: FastEthernet0/0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to FE80::CE01:1BFF:FE29:0, timeout is 2 seconds:
Packet sent with a source address of FE80::CE00:1BFF:FE29:0
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/4/8 ms
</code></pre>

Pinging each other is no issue. Is OSPFv3 running?

<pre><code><strong>R1#show ipv6 protocols 
</strong>IPv6 Routing Protocol is "connected"
IPv6 Routing Protocol is "static"
IPv6 Routing Protocol is "ospf 1"
  Interfaces (Area 0):
    FastEthernet0/0
</code></pre>

```
R2#show ipv6 protocols 
IPv6 Routing Protocol is "connected"
IPv6 Routing Protocol is "static"
IPv6 Routing Protocol is "ospf 1"
  Interfaces (Area 0):
    FastEthernet0/0
```

OSPFv3 has been enabled on the interfaces, still we don’t have any neighbors:

<pre><code><strong>R1#show ipv6 ospf neighbor 
</strong></code></pre>

<pre><code><strong>R2#show ipv6 ospf neighbor 
</strong></code></pre>

Unfortunately we don’t have any neighbors. Let’s enable a debug:

<pre><code><strong>R1#debug ipv6 ospf hello 
</strong>  OSPFv3 hello events debugging is on
</code></pre>

Becoming OSPFv3 neighbors starts with a hello packet, let’s see what we discover:

```
R1# OSPFv3: Mismatched hello parameters from FE80::CE01:1BFF:FE29:0
    OSPFv3: Dead R 36 C 40, Hello R 9 C 10
    OSPFv3: Send hello to FF02::5 area 0 on FastEthernet0/0 from FE80::CE
```

There we go…we see that there is a mismatch in the hello parameters. R1 is configured to send hello packets each 10 seconds and R2 is configured to send them every 9 seconds. The dead timer also has a mismatch. Here’s what the timers look like:

<pre><code><strong>R1#show ipv6 ospf 1 interface fa0/0 | include intervals
</strong>  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
</code></pre>

<pre><code><strong>R2#show ipv6 ospf 1 interface fa0/0 | include intervals
</strong>  Timer intervals configured, Hello 9, Dead 36, Wait 36, Retransmit 5
</code></pre>

We can verify the timers by using the show ipv6 ospf interface command. Let’s make sure they match:

<pre><code><strong>R2(config)#interface fa0/0
</strong><strong>R2(config-if)#ipv6 ospf hello-interval 10
</strong></code></pre>

Let’s change the hello timer back to 10 seconds, this also changes the dead timer interval. After a few seconds you’ll see this:

```
R1# %OSPFv3-5-ADJCHG: Process 1, Nbr 2.2.2.2 on FastEthernet0/0 from LOADING to FULL, Loading Done
```

```
R2# %OSPFv3-5-ADJCHG: Process 1, Nbr 1.1.1.1 on FastEthernet0/0 from LOADING to FULL, Loading Done
```

That’s looking good, the routers have become OSPFv3 neighbors:

<pre><code><strong>R1#show ipv6 ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
2.2.2.2           1   FULL/DR         00:00:33    4           FastEthernet0/0
</code></pre>

<pre><code><strong>R2#show ipv6 ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
1.1.1.1           1   FULL/BDR        00:00:31    4           FastEthernet0/0
</code></pre>

Another problem bites the dust!

**Lesson Learned: OSPFv3 for IPv6 has the same requirements to form a neighbor adjacency as OSPFv2 for IPv4. Apply your “IPv4 OSPF” knowledge to solve neighbor adjacency issues.**

## OSPFv3 over Frame-Relay

Here’s something different for you. We are still working on troubleshooting OSPFv3 neighbor adjacencies but we will use a different topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/ospfv3-over-frame-relay.png" alt=""><figcaption></figcaption></figure>

In the scenario above we have a frame-relay setup. There’s a single PVC between R1 and R2. R1 has been configured to use 2001:12::1 and R2 is using 2001:12::2. For some reason the OSPFv3 neighbor adjacency is not working…let’s troubleshoot!

What’s the best place to start troubleshooting a problem like this? There are two options:

* Start in the middle of the OSI-model and dive into the OSPFv3 configuration right away.
* Start at the bottom of the OSI-model and check if the frame-relay configuration is properly configured.

Personally I like a structured approach and start at the bottom of the OSI model and work my way up. In this case that means we have to check if the interfaces are up, frame-relay encapsulation has been configured, if the PVC is working, if we have a valid frame-relay map, if we can ping each other and then move on to OSPFv3.

If you start at the bottom you’ll find the problem eventually but it might be a bit more time-consuming sometimes. Just to try something different I’ll start in the middle of the OSI-model this time and we’ll check the OSPFv3 configuration first:

<pre><code><strong>R1#show ipv6 ospf neighbor 
</strong></code></pre>

<pre><code><strong>R2#show ipv6 ospf neighbor 
</strong></code></pre>

I can confirm that there are no neighbor adjacencies. Let’s see if OSPFv3 is active:

<pre><code><strong>R1#show ipv6 protocols 
</strong>IPv6 Routing Protocol is "connected"
IPv6 Routing Protocol is "static"
IPv6 Routing Protocol is "ospf 1"
  Interfaces (Area 0):
    Serial0/0
</code></pre>

<pre><code><strong>R2#show ipv6 protocols 
</strong>IPv6 Routing Protocol is "connected"
IPv6 Routing Protocol is "static"
IPv6 Routing Protocol is "ospf 1"
  Interfaces (Area 0):
    Serial0/0
</code></pre>

You can see that OSPFv3 has been enabled for the serial 0/0 interfaces. Let’s take a closer look at the OSPFv3 configuration:

<pre><code><strong>R1#show ipv6 ospf interface serial 0/0
</strong>Serial0/0 is up, line protocol is up 
  Link Local Address FE80::9CD7:2EFF:FEF0:99FA, Interface ID 4
  Area 0, Process ID 1, Instance ID 0, Router ID 1.1.1.1
  Network Type NON_BROADCAST, Cost: 64
  Transmit Delay is 1 sec, State DR, Priority 1 
  Designated Router (ID) 1.1.1.1, local address FE80::9CD7:2EFF:FEF0:99FA
  No backup designated router on this network
  Timer intervals configured, Hello 30, Dead 120, Wait 120, Retransmit 5
</code></pre>

<pre><code><strong>R2#show ipv6 ospf interface serial 0/0
</strong>Serial0/0 is up, line protocol is up 
  Link Local Address FE80::9CD7:2EFF:FEF0:99FA, Interface ID 4
  Area 0, Process ID 1, Instance ID 0, Router ID 2.2.2.2
  Network Type NON_BROADCAST, Cost: 64
  Transmit Delay is 1 sec, State DR, Priority 1 
  Designated Router (ID) 2.2.2.2, local address FE80::9CD7:2EFF:FEF0:99FA
  No backup designated router on this network
  Timer intervals configured, Hello 30, Dead 120, Wait 120, Retransmit 5
</code></pre>

This is something that is worth a closer look. The network type and timer intervals match. The network type is “NON\_BROADCAST” which means that we have to configure our neighbors ourselves. Has this been done? Let’s take a look:

<pre><code><strong>R1#show ipv6 ospf neighbor 
</strong></code></pre>

<pre><code><strong>R2#show ipv6 ospf neighbor 
</strong></code></pre>

Nobody configured any neighbors otherwise they would show up in the output above. Let’s fix this, first I’ll have to check what the link-local addresses are:

<pre><code><strong>R1#show ipv6 interface brief 
</strong>Serial0/0                  [up/up]
    FE80::9CD8:2EFF:FEF0:99FA
</code></pre>

<pre><code><strong>R2#show ipv6 interface brief 
</strong>Serial0/0                  [up/up]
    FE80::9CD7:2EFF:FEF0:99FA
</code></pre>

Here you can see the link-local addresses that we need to use. Let’s configure the neighbors:

<pre><code><strong>R1(config)#interface serial 0/0
</strong><strong>R1(config-if)#ipv6 ospf neighbor FE80::9CD7:2EFF:FEF0:99FA
</strong></code></pre>

<pre><code><strong>R2(config)#interface serial 0/0
</strong><strong>R2(config-if)#ipv6 ospf neighbor FE80::9CD8:2EFF:FEF0:99FA
</strong></code></pre>

This is how we configure the neighbors ourselves. Let’s see if this solves our problem:

<pre><code><strong>R1#show ipv6 ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
N/A               0   ATTEMPT/DROTHER 00:01:00    0               Serial0/0
</code></pre>

<pre><code><strong>R2#show ipv6 ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
N/A               0   ATTEMPT/DROTHER 00:00:39    0               Serial0/0
</code></pre>

Too bad…I want to see “FULL” but I’m only seeing “ATTEMPT” here. My OSPFv3 configuration looks fine to me so this would be a good moment to move further down the OSI model. Let’s try a quick ping between the two routers:

<pre><code><strong>R1#ping FE80::9CD7:2EFF:FEF0:99FA
</strong>Output Interface: Serial0/0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to FE80::9CD7:2EFF:FEF0:99FA, timeout is 2 seconds:
Packet sent with a source address of FE80::9CD8:2EFF:FEF0:99FA
.....
Success rate is 0 percent (0/5)
</code></pre>

I’m unable to ping between the link-local addresses. Now I know that layer 3 of the OSI-model is not working, let’s dive deeper…

<pre><code><strong>R1#show frame-relay map 
</strong></code></pre>

<pre><code><strong>R2#show frame-relay map 
</strong></code></pre>

There seems to be no frame-relay map. We need this in order to bind the DLCI to the IPv6 addresses. Normally Inverse ARP takes care of this but not this time. It’s probably disabled (or the interface is not operational). Let’s check if the PVCs are active:

<pre><code><strong>R1#show frame-relay pvc | include ACTIVE
</strong>DLCI = 102, DLCI USAGE = UNUSED, PVC STATUS = ACTIVE, INTERFACE = Serial0/0
</code></pre>

<pre><code><strong>R2#show frame-relay pvc | include ACTIVE
</strong>DLCI = 201, DLCI USAGE = UNUSED, PVC STATUS = ACTIVE, INTERFACE = Serial0/0
</code></pre>

We’ll check if the PVC is operational and what the DLCI number is. It’s active on both sides and you can see the DLCI numbers, let’s create some frame-relay maps:

<pre><code><strong>R1(config)#interface serial 0/0
</strong><strong>R1(config-if)#frame-relay map ipv6 FE80::9CD7:2EFF:FEF0:99FA 102
</strong></code></pre>

<pre><code><strong>R2(config)#interface serial 0/0
</strong><strong>R2(config-if)#frame-relay map ipv6 FE80::9CD8:2EFF:FEF0:99FA 201
</strong></code></pre>

I’ll map the link-local addresses to the DLCI numbers. After a few seconds we’ll see this:

```
R1# %OSPFv3-5-ADJCHG: Process 1, Nbr 2.2.2.2 on Serial0/0 from LOADING to FULL, Loading Done
```

```
R2# %OSPFv3-5-ADJCHG: Process 1, Nbr 1.1.1.1 on Serial0/0 from LOADING to FULL, Loading Done
```

This is looking good! Let’s verify our work:

<pre><code><strong>R1#show ipv6 ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
2.2.2.2           1   FULL/DR         00:01:47    4               Serial0/0
</code></pre>

<pre><code><strong>R2#show ipv6 ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
1.1.1.1           1   FULL/BDR        00:01:58    4               Serial0/0
</code></pre>

Problem solved!

**Lesson learned: Check the OSPFv3 network type and configure the neighbors using the link-local addresses. Also make sure you have the correct frame-relay maps.**

That’s all I have for now, keep in mind that most of what you know about OSPF (version 2) can also be applied to troubleshooting OSPFv3. If you have any questions, feel free to leave a comment!
