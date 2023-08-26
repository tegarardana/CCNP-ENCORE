# EIGRP Neighbor Adjacency

In another lesson, I explained the different [EIGRP packets](https://networklessons.com/cisco/ccnp-encor-350-401/eigrp-packets-explained) and their function. In this lesson, we’ll take a close look at the EIGRP neighbor adjacency to see what exactly happens when EIGRP routers become neighbors. This is what happens when you enable EIGRP on two routers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-neighbor-hello.png" alt=""><figcaption></figcaption></figure>

We have two routers called R1 and R2, and they are configured for EIGRP. As soon as we enable it for the interface, they will start sending hello packets. In this example, R1 is the first router to send a hello packet.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-neighbor-hello-update.png" alt=""><figcaption></figcaption></figure>

As soon as R2 receives the hello packet from R1 it will respond by sending update packets that contain all the routing information that it has in its routing table. The only routes not sent on this interface are the ones that R2 learned on this interface because of split horizon. The update packet R2 will send has the initialization bit set, so we know this is the “initialization process.”  At this moment, there is still no neighbor adjacency until R2 has sent a hello packet to R1.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-neighbor-hello-update-hello.png" alt=""><figcaption></figcaption></figure>

R1 is, of course, not the only one sending hello packets. As soon as R2 sends a hello packet to R1 we can continue to set up a neighbor adjacency.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-neighbor-ack.png" alt=""><figcaption></figcaption></figure>

After both routers have exchanged hello packets, we will establish the neighbor adjacency. R1 will send an ACK to let R2 know he received the update packets. The routing information in the update packets will be saved in the EIGRP topology table.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-neighbor-update-1.png" alt=""><figcaption></figcaption></figure>

R2 is anxious to receive routing information as well, so R1 will send update packets to R2, which will save this information in its EIGRP topology table.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-neighbor-ack-return.png" alt=""><figcaption></figcaption></figure>

After receiving the update packets, R2 will send an ACK back to R1 to let him know everything is ok.

## Configuration

Want to see what this looks like on a real router? Let’s use the following topology and see what happens:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-neighbor-adjacency-lab.png" alt=""><figcaption></figcaption></figure>

This is the topology I’m going to use to configure EIGRP. My goal is to have full connectivity, and here are the configurations:

<pre><code><strong>R1(config)#router eigrp 1
</strong><strong>R1(config-router)#no auto-summary 
</strong><strong>R1(config-router)#network 1.1.1.0 0.0.0.255
</strong><strong>R1(config-router)#network 192.168.12.0
</strong><strong>R1(config-router)#exit
</strong></code></pre>

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)#no auto-summary 
</strong><strong>R2(config-router)#network 2.2.2.0 0.0.0.255
</strong><strong>R2(config-router)#network 192.168.12.0
</strong><strong>R2(config-router)#exit
</strong></code></pre>

Let’s break this one down. `Router eigrp 1` will start up EIGRP using AS (autonomous system) number 1. This number has to match on both routers, or we won’t become EIGRP neighbors.

`No auto-summary` is needed because, by default EIGRP will behave like a classful routing protocol which means it won’t advertise the subnet mask along the routing information. In this case, that means that 1.1.1.0/24 and 2.2.2.0/24 will be advertised as 1.0.0.0/8 and 2.0.0.0/8. Disabling auto-summary will ensure EIGRP sends the subnet mask along.

`Network 1.1.1.0 0.0.0.255` means I’m advertising the 1.1.1.0 network with wildcard 0.0.0.255. If I don’t specify the wildcard, you’ll find “`network 1.0.0.0`” in your configuration. Does it matter? Yes and no. The same thing applies to “network 2.2.2.0 /24”. It will work but it also means that every interface that falls within the 1.0.0.0/8 or 2.0.0.0/8 range is going to run EIGRP. Network 192.168.12.0 without a wildcard mask is fine since I’m using a /24 on this interface which is Class C.

If you are working on a lab and are lazy, you can also type in network 0.0.0.0, which will activate EIGRP on all of your interfaces.

Let’s do a debug on R2 to see what is going on:

<pre><code><strong>R2#debug eigrp packets ?
</strong>  SIAquery  EIGRP SIA-Query packets
  SIAreply  EIGRP SIA-Reply packets
  ack       EIGRP ack packets
  hello     EIGRP hello packets
  ipxsap    EIGRP ipxsap packets
  probe     EIGRP probe packets
  query     EIGRP query packets
  reply     EIGRP reply packets
  request   EIGRP request packets
  retry     EIGRP retransmissions
  stub      EIGRP stub packets
  terse     Display all EIGRP packets except Hellos
  update    EIGRP update packets
  verbose   Display all EIGRP packets
  &#x3C;cr>
</code></pre>

As you can see, we have many debug options for EIGRP. I want to see the hello packets:

<pre><code><strong>R2#debug eigrp packets hello 
</strong>EIGRP Packets debugging is on
    (HELLO)

R2# EIGRP: Received HELLO on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/0
</code></pre>

This is looking good. It seems we have received a hello packet from R1. R2 sends a hello packet to R1 as well:

```
R2# EIGRP: Sending HELLO on FastEthernet0/0
  AS 1, Flags 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0
```

A little later, we have an EIGRP neighbor adjacency:

```
R1# %DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 192.168.12.2 (FastEthernet0/0) is up: new adjacency
```

```
R2# %DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 192.168.12.1 (FastEthernet0/0) is up: new adjacency
```

If you look closely at the debug, you can see something interesting:

<pre><code><strong>R2# EIGRP: Sending HELLO on Loopback0
</strong>AS 1, Flags 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0

<strong>EIGRP: Received HELLO on Loopback0 nbr 2.2.2.2
</strong>AS 1, Flags 0x0, Seq 0/0 idbQ 0/0
</code></pre>

It seems R2 is schizophrenic and sending and receiving hello packets on its loopback interface.

This behavior is normal because the network command does two things:

* Send EIGRP packets on the interface that falls within the network command range.
* Advertise the network that is configured on the interface in EIGRP.

So what do you have to do when you want to advertise a network without sending EIGRP packets on the interface and forming EIGRP neighbors?

You use the `passive interface` command:

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)#passive-interface loopback 0
</strong></code></pre>

This will advertise the 2.2.2.0/24 network on R2’s loopback 0 interface **without** sending EIGRP packets to the loopback. If you have to configure an ISP router with 50+ interfaces, you probably don’t want to type in this command on each of those interfaces. You can also configure passive-interface default and only activate the interfaces you want to run EIGRP on. Here is an example:

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)#passive-interface default 
</strong><strong>R2(config-router)#no passive-interface fastEthernet 0/0
</strong></code></pre>

Let’s look at a debug of some EIGRP update packets. First, we enable the debug:

<pre><code><strong>R2#debug eigrp packets update 
</strong>EIGRP Packets debugging is on
    (UPDATE)
</code></pre>

And now we clear the neighbor adjacency:

<pre><code><strong>R1#clear ip eigrp neighbors
</strong></code></pre>

This is what you’ll see on R1:

```
R1# %DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 192.168.12.2 (FastEthernet0/0) is down: Interface Goodbye received
```

You can see that EIGRP neighbors tell each other when they are clearing the neighbor adjacency. A short while later, we reestablish the neighbor adjacency:

```
R1# %DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 192.168.12.2 (FastEthernet0/0) is up: new adjacency
```

R1 is back in business, and our EIGRP neighbor adjacency has been re-established. Once we have a neighbor adjacency, we’ll exchange UPDATE packets:

```
R2#
EIGRP: Enqueueing UPDATE on FastEthernet0/0 nbr 192.168.12.1 iidbQ un/rely 0/1 peerQ un/rely 0/0
EIGRP: Received UPDATE on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x1, Seq 13/0 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/1
EIGRP: Sending UPDATE on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x1, Seq 13/13 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/1
```

Above, you can see R2 is putting an update packet in its queue for R1. R2 is also receiving an update packet from R1 and afterward sending its own update packet. You’ll see a couple of update packets flying back and forth.

<pre><code><strong>R2#debug eigrp packets ack
</strong>EIGRP Packets debugging is on
    (ACK)
</code></pre>

We can also check out some acknowledgments by debugging ack packets. Let’s reset the neighbor adjacency again:

<pre><code><strong>R1#clear ip eigrp neighbors
</strong></code></pre>

Here’s what you’ll get:

```
R2a#
EIGRP: Received ACK on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/17 idbQ 0/0 iidbQ un/rely 0/1 peerQ un/rely 0/1
EIGRP: Received ACK on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/18 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/1
EIGRP: Enqueueing ACK on FastEthernet0/0 nbr 192.168.12.1
  Ack seq 18 iidbQ un/rely 0/0 peerQ un/rely 1/2
EIGRP: Sending ACK on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/18 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 1/2
EIGRP: Enqueueing ACK on FastEthernet0/0 nbr 192.168.12.1
  Ack seq 19 iidbQ un/rely 0/0 peerQ un/rely 1/2
EIGRP: Sending ACK on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/19 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 1/2
EIGRP: Received ACK on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/20 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/1
```

These are the ack packets in response to some of the update packets. If you want to see the whole process, we can combine some debugging:

<pre><code><strong>R2#debug eigrp packets 
</strong>EIGRP Packets debugging is on
    (UPDATE, REQUEST, QUERY, REPLY, HELLO, IPXSAP, PROBE, ACK, STUB, SIAQUERY, SIAREPLY)
</code></pre>

This is how you enable debugging for all EIGRP packets. Reset the neighbor adjacency one more:

<pre><code><strong>R1#clear ip eigrp neighbors
</strong></code></pre>

And you’ll see plenty of debugging information:

```
R2#
EIGRP: Sending HELLO on FastEthernet0/0
  AS 1, Flags 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0
EIGRP: Received HELLO on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/0 idbQ 0/0
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 192.168.12.1 (FastEthernet0/0) is up: new adjacency
EIGRP: Enqueueing UPDATE on FastEthernet0/0 nbr 192.168.12.1 iidbQ un/rely 0/1 peerQ un/rely 0/0
EIGRP: Sending HELLO on FastEthernet0/0
  AS 1, Flags 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/1
EIGRP: Requeued unicast on FastEthernet0/0
EIGRP: Received UPDATE on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x1, Seq 25/0 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/1
EIGRP: Sending UPDATE on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x1, Seq 25/25 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/1
EIGRP: Received ACK on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/25 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/1
EIGRP: Enqueueing UPDATE on FastEthernet0/0 iidbQ un/rely 0/1 serno 1-2
EIGRP: Enqueueing UPDATE on FastEthernet0/0 nbr 192.168.12.1 iidbQ un/rely 0/0 peerQ un/rely 0/0 serno 1-2
EIGRP: Sending UPDATE on FastEthernet0/0
  AS 1, Flags 0x8, Seq 26/0 idbQ 0/0 iidbQ un/rely 0/0 serno 1-2
EIGRP: Enqueueing UPDATE on FastEthernet0/0 nbr 192.168.12.1 iidbQ un/r
R2#ely 0/1 peerQ un/rely 0/1
EIGRP: Requeued unicast on FastEthernet0/0
EIGRP: Received ACK on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/26 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/2
EIGRP: FastEthernet0/0 multicast flow blocking cleared
EIGRP: Sending UPDATE on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x8, Seq 27/25 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/1
EIGRP: Received ACK on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/27 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/1
EIGRP: Received UPDATE on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x8, Seq 26/0 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/0
EIGRP: Enqueueing ACK on FastEthernet0/0 nbr 192.168.12.1
  Ack seq 26 iidbQ un/rely 0/0 peerQ un/rely 1/0
EIGRP: Sending ACK on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/26 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 1/0
EIGRP: Enqueueing UPDATE on FastEthernet0/0 iidbQ un/rely 0/1 serno 9-9
EIGRP: Enqueueing UPDATE on FastEthernet0/0 nbr 192.168.12.1 iidbQ un/rely 0/0 peerQ un/rely 0/0 serno 9-9
EIGRP: Received UPDATE on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x8, Seq 27/27 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/1
EIGRP: Enqueueing ACK on FastEthernet0/0 nbr 192.168.12.1
  Ack seq 27 iidbQ un/rely 0/0 peerQ un/rely 1/1
EIGRP: Sending ACK on FastEthernet0/0 nbr 192.168.12.1
  AS 1, Flags 0x0, Seq 0/27 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 1/1
```

This will show you the entire process of sending hello packets to each other, becoming EIGRP neighbors, exchanging updates, and sending acks. That’s the end of my story. I hope this has been helpful for you in understanding the EIGRP neighbor adjacency process. If you have any questions or comments, please leave a message.
