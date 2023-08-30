# Multicast Routing

The main goal of a router is to route packets. In other words: when it receives an IP packet it has to look at the destination address, check the routing table and figure out the next hop where to forward the IP packet to. We use routing protocols to learn different networks and to fill the routing table.

Here’s an illustration to visualize this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/r1-r2-unicast-routing-table.png" alt=""><figcaption></figcaption></figure>

Above we see R1 who wants to deliver an IP packet to destination 2.2.2.2. It checks its routing table, finds an OSPF entry with 192.168.12.2 as the next hop (R2). R1 is now able to forward the IP packet towards the destination.

Everything I explained above applies to **unicast traffic**. What about multicast traffic? Let’s look at another example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/r1-r2-video-server-multicast-routing.png" alt=""><figcaption></figcaption></figure>

Above we see R1 who receives a multicast packet from some video server, the destination address is 239.1.1.1. The routing table however is a **unicast routing table**. There’s no information about any multicast addresses in there. Our router will have no idea where to forward it’s multicast traffic to…

## Multicast Routing Protocols

To route our multicast traffic, we need to use a **multicast routing protocol**. There are two types of multicast routing protocols:

* Dense Mode
* Sparse Mode

We’ll discuss both types, we’ll start with dense mode.

### Dense Mode

Dense mode multicast routing protocols are used for networks where **most subnets in your network should receive the multicast traffic**. When a router receives the multicast traffic, it will flood it on all of its interfaces except the interface where it received the multicast traffic on. Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-dense-mode-flooding.png" alt=""><figcaption></figcaption></figure>

Above we have a video server sending multicast traffic to R1. When R1 receives these packets, it will flood them on all of its interfaces. R2 and R3 will do the same so our two hosts (H2 and H3) will receive the multicast traffic. In the example above both of our hosts are interested in the multicast traffic but what if there are hosts that don’t want to receive it?

A multicast router can tell its neighbor that it doesn’t want to receive the multicast traffic anymore. This happens when:

* The router doesn’t have any downstream neighbors that require the multicast traffic.
* The router doesn’t have any hosts on its directly connected interface that require the multicast traffic.

Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-dense-mode-pruning.png" alt=""><figcaption></figcaption></figure>

Above we see R1 that receives the multicast traffic from our video server. It floods this multicast traffic to R2 and R3 but these two routers don’t have any interest in the multicast traffic. They will send a **prune message** to signal R1 that it should no longer forward the multicast traffic.

There are a number of dense mode routing protocols:

* DVMRP (Distance Vector Multicast Routing Protocol)
* MOSPF (Multicast OSPF)
* PIM Dense Mode

PIM (Protocol Independent Multicast) is the most popular multicast routing protocol which we will discuss in other multicast lessons.

### RPF (Reverse Path Forwarding)

Multicast routing is vulnerable to routing loops. One simple loop-prevention mechanism is that routers will never forward multicast packets on the interface **where they received the packet on**. There is one additional check however called **RPF (Reverse Path Forwarding)**. Take a look at the example below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-dense-mode-loop.png" alt=""><figcaption></figcaption></figure>

Above we have R1 which receives a multicast packet which is flooded on all interfaces except the interface that connects to the video server. I’m only showing the packet that is flooded towards R3 here:

* R1 floods the packet to R3.
* R3 floods the packet to R2.
* R2 floods it back to R1.

We now have a multicast routing loop. We can prevent this by implementing the RPF check:

When a router receives a multicast packet on an interface, it looks at the source IP address and does two checks:

* Do we have an entry that matches the source address in the unicast routing table?
* If so, what interface do we use to reach that source address?

When the multicast packet is received on the interface that matches the information from the unicast routing table, it passes the RPF check and we **accept the packet**. When it fails the RPF check, we **drop the packet**.

Let me visualize this with an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-dense-rpf-check.png" alt=""><figcaption></figcaption></figure>

Above we see R1 which floods the multicast traffic to R2 and R3. R2 also floods it to R3.

R3 will now perform a RPF check for both multicast packets. It sees the source address is 192.168.1.100 and checks the unicast routing table. It finds an OSPF entry for 192.168.1.0/24 that points to R1.

The packet that it receives from R1 will pass the RPF check since we receive it on the Fa0/0 interface, the one it receives from R2 doesn’t. The multicast packet from R2 **will be dropped**.

R3 will then flood the multicast packet towards R2 who will also do a RPF check. It will drop this packet since R2 uses its interface towards R1 to reach 192.168.1.100.

Another way to look at this is that the RPF check ensures that we only accept multicast packets from the **shortest path**. Multicast packets that travel **longer paths are dropped**. For R3 the shortest path to 192.168.1.100 is through R1.

#### Sparse Mode

At this moment you might be thinking that dense mode is very inefficient with its flooding of multicast traffic. When you only have a few receivers on your network then yes, you will be wasting a lot of bandwidth and resources on your routers.

The alternative is **sparse mode** which is far more efficient. Sparse mode multicast routing protocols **only forward the multicast traffic when another router requests it**. It’s the complete opposite of dense mode:

* Dense mode floods multicast traffic until a router **asks you to stop**.
* Sparse mode sends multicast traffic only **when a router requests it**.

Requesting multicast traffic sounds great but it introduces one problem…where are you going to send your request to? With dense mode, you will receive the traffic whether you like it or not. With sparse mode…you have no idea where the multicast traffic should come from.

To fix this issue, sparse mode uses a special router called the **RP (Rendezvous Point)**. All multicast traffic is forwarded to the RP and when other routers want to receive it, they’ll have to find their way towards the RP. Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/pim-sparse-mode-rp-receiving-multicast-traffic.png" alt=""><figcaption></figcaption></figure>

Above we see R1 which is the RP for our network. It’s receiving multicast traffic from the video server but at the moment nobody is interested in it. R1 will not send any multicast traffic on the network at this moment.

Here’s when a router will request multicast traffic from another router:

* When the router has **received an IGMP join message from a host** that is directly connected.
* When the router receives a request from another router.

Here’s an illustration:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/pim-sparse-mode-rp-pim-join.png" alt=""><figcaption></figcaption></figure>

Let me explain what we have above:

1. The video server is sending multicast traffic to R1.
2. R1 is the RP for this network.
3. H3 wants to receive this multicast traffic so it sends an IGMP join message for multicast group 239.1.1.1.
4. R3 receives the IGMP join and will request R1 to start sending the multicast traffic. In this example, R3 has sent a PIM join message. (we’ll talk about PIM later).

R1 will now start forwarding the multicast traffic to R3:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/pim-sparse-mode-rp-forward-traffic.png" alt=""><figcaption></figcaption></figure>

Our host is now able to receive the multicast traffic, we didn’t waste any resources by sending unwanted multicast traffic towards R2 and H2.

When using sparse mode, **all routers need to know the IP address of the RP**. When we take a closer look at PIM sparse mode, you will learn that the RP address can be configured statically or learned dynamically.

### Multicast Scoping

When you implement multicast in your network you might want to restrict it somehow. There are two options for this:

* TTL (Time to Live) scoping
* Administrative scoping

We’ll take a look at both options.

### TTL Scoping

Like “normal” IP packets, each multicast packet has a TTL (Time to Live) field. When a router receives a multicast packet, it decreases the TTL by one before it forwards the packet on other interfaces. If you want you can set a certain threshold for the TTL value. When the TTL value of the IP packet is equal or greater than the threshold then the packet will be forwarded, otherwise it will be discarded. Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-ttl-threshold.png" alt=""><figcaption></figcaption></figure>

The router above is receiving multicast packets from the video server with a TTL of 10. Before forwarding the packet, it reduces the TTL by one to 9.

Its interfaces are configured with a TTL threshold. It won’t forward any multicast packets on its Fa0/1 interface since the TTL (9) is below the configured threshold (20). The packet can be forwarded on its Fa0/0 interface since the TTL (9) is higher than the configured threshold (5).

TTL scoping however is a poor method to restrict your multicast traffic:

1. Your application has to support configuring a custom TTL value.
2. Estimating a useful TTL value is difficult since it depends on the number of routers between the source and each receiver in your network.
3. It applies to all multicast traffic so if you want to make an exception, you need to configure your applications to use different TTL values.

### Administrative Scoping

Multicast uses the 239.0.0.0 – 239.255.255.255 range as private addresses, similar to the RFC1918 addresses we use for unicast routing.

If you really want to restrict multicast traffic on your network then you can filter it using access-lists. Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-access-list-filtering.png" alt=""><figcaption></figcaption></figure>

Above we see R1 with an access-list on its Fa0/1 interface. The access-list has a deny statement that drops all traffic destined for 239.0.0.0 – 239.255.255.255. This is a simple but effective method to deny multicast traffic on your network.

## Conclusion

You have now learned the basics of multicast routing and how it is different from unicast routing. Dense mode multicast routing protocols flood all traffic until a neighbor router tells you to stop. The RPF check is used to ensure we don’t get routing loops on our network. Sparse mode multicast routing uses a RP (Rendezvous point) and works with a “pull” mechanism. Multicast traffic is only forwarded when we request it.

The most popular multicast routing protocol is PIM (Protocol Independent Multicast) which supports dense and sparse mode. We’ll take a closer look at these two options in upcoming lessons.

If you have any questions, feel free to leave a comment.

\
