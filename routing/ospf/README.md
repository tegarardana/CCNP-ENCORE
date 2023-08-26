# OSPF

## Introduction to OSPF

OSPF is a link-state routing protocol, and it’s one of the routing protocols you need to understand if you want to do the Cisco CCNA, CCNP, or CCIE R\&S exam(s). In this lesson, I’ll explain the basics of OSPF to you and you will learn how and why it works.

<div align="left">

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/navigation-system-300x225.jpg" alt=""><figcaption></figcaption></figure>

</div>

I don’t know about you, but I love my navigation system. The good thing about them is you can just drive and there is no need to look for traffic signs, the bad thing is that I’m absolutely lost when it’s not working. I’m bad at reading maps (or maybe I don’t like them) and if I had to find my way to some street in any big city I’d be doomed.

Link-state routing protocols are like your navigation system, they have a complete map of the network. If you have a full map of the network you can calculate the shortest path to all the different destinations out there. This is cool because if you know about all the different paths, it’s impossible to get a loop since you know everything! The downside is that this is more CPU intensive than a distance vector routing protocol. It’s just like your navigation system…if you calculate a route from New York to Los Angeles, it’s going to take a bit longer than when you calculate a route from one street to another street in the same city.

Let’s take a good look at link-state to see what it exactly means:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/routers-lsdb-lsa.png" alt=""><figcaption></figcaption></figure>

* **Link**: That’s the interface of our router.
* **State**: Description of the interface and how it’s connected to neighbor routers.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/lsa-into-lsdb.png" alt=""><figcaption></figcaption></figure>

Link-state routing protocols operate by sending link-state advertisements (LSA) to all other link-state routers.

All the routers need to have these link-state advertisements so they can build their link-state database or LSDB. Basically, all the link-state advertisements are a piece of the puzzle that builds the LSDB.

If you have a lot of OSPF routers, it might not be very efficient that each OSPF router floods its LSAs to all other OSPF routers. Let me show you an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-multi-access-network.png" alt=""><figcaption></figcaption></figure>

Above, we have a network with 8 OSPF routers connected on a switch. Each of those routers is going to become OSPF neighbors with all of the other routers…sending hello packets, flooding LSAs, and building the LSDB. This is what will happen:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-full-mesh.png" alt=""><figcaption></figcaption></figure>

We will get a full mesh of OSPF neighbors. Each router will flood LSAs to all other routers so we will have a lot of OSPF traffic. Is there any way to make this a bit more efficient?

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-designated-router.png" alt=""><figcaption></figcaption></figure>

What if all our OSPF routers would just send their stuff to a single OSPF router who will then forward it to all the other OSPF routers? All our OSPF routers will know about all the routing information out there, but we will have far less OSPF traffic. OSPF uses something called a DR (Designated Router). All our OSPF routers will only form a “full” neighbor adjacency with the DR and not with all the other routers.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-backup-designated-router.png" alt=""><figcaption></figcaption></figure>

Since bad stuff can happen to our networks, we want to have a backup for our DR. If it crashes, the BDR (Backup Designated Router) will take over. All our OSPF routers will only form full neighbor adjacencies with the DR and BDR and not with all other routers. This sounds efficient, right?

We only use a DR/BDR on a multi-access network. An example of a multi-access network is using a switch. There is no need to do this on a point-to-point link. There is only one other router on the other side so there is no reason to select a DR/BDR.

The LSDB is our full picture of the network, in network terms we call this the topology.

You could compare the LSDB to having a full map of your country.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-link-state-database-1.png" alt=""><figcaption></figcaption></figure>

Once every router has a complete map, we can start calculating the shortest path to all the different destinations by using the shortest-path first (SPF) algorithm. The BEST information goes into the routing table. Calculating the shortest path is like using your navigation system. It will look at the map and look at all the different ways of getting to the destination and only show you the best way of getting there.

There is only one link-state routing protocol we are going to discuss which is OSPF (Open Shortest Path First). There is another link-state routing protocol called IS-IS.

Enough of my link-state routing protocol introduction let’s take a look at OSPF and see how it operates.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-single-area.png" alt=""><figcaption></figcaption></figure>

OSPF works with the concepts of areas, and by default, you will always have a single area, normally, this is area 0 or also called the backbone area.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-multi-area.png" alt=""><figcaption></figcaption></figure>

You can have multiple areas however, as in the picture above, we have areas 1,2, and 3. All of these areas must connect to the backbone area. If you want to go from area 1 to area 2, you must go through the backbone area to get there. It’s impossible to go from area 1 directly to area 2; you always have to pass the backbone area! Same thing if you want to go from area 3 to area 2…you must cross the backbone area.

So why do we work with areas? Remember what I just explained about your navigation system? If you tell your navigation system to calculate from New York to Los Angeles, it will take much longer than calculating a route from one street to another street in the same city. This calculation is called the shortest path first, or SPF algorithm and the same thing applies to OSPF. Our routers only have a full picture of the network topology within the area. The smaller your map, the faster your SPF algorithm works!

Keep in mind the SPF algorithm is from the ’70s, and OSPF was invented somewhere in the ’80s…we didn’t have fancy high GHz multi-core CPUs back then.

In the picture, you also see on the top right something called “other routing domain.” This could be another network running another routing protocol (perhaps RIP), and it’s possible to import and export routes from RIP into OSPF or the other way around, this is called redistribution.

* Routers in the backbone area (area 0) are called backbone routers.
* Routers between 2 areas (like the one between area 0 and area 1) are called area border routers or ABR.
* Routers that run OSPF and are connected to another network that runs another routing protocol (for example, RIP) are called autonomous system border routers or ASBR.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-hello-packet-1.png" alt=""><figcaption></figcaption></figure>

OSPF works differently than RIP or EIGRP, first of all, it’s a link-state compared to a distance vector (RIP), but it also doesn’t just send the link-state advertisements around. Routers have to become neighbors first. Once we have become neighbors, we are going to exchange link-state advertisements.\
Once you configure OSPF, your router will start sending hello packets. If you also receive hello packets from the other router, you will become neighbors.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-hello-packet-attributes.png" alt=""><figcaption></figcaption></figure>

There’s one catch, however, there are a couple of fields in the hello packet, and many of them have to match otherwise, you won’t become neighbors. Let’s walk through the items in the hello packet and see what they are about:

* Router ID: Each OSPF router needs to have a unique ID which is the highest IP address on any active interface. More about this later.
* Hello / Dead Interval: Every X seconds, we will send a hello packet. If we don’t hear any hello packets from our network for X seconds, we declare you “dead,” and we are no longer neighbors. These values have to match on both sides to become neighbors.
* Neighbors: All other routers who are your neighbors are specified in the hello packet.
* Area ID: This is the area you are in. This value has to match on both sides to become neighbors.
* Router Priority: This value is used to determine who will become designated or backup designated routers.
* DR and BDR IP address: Designated and Backup Designated router.
* Authentication password: You can use clear text and MD5 authentication for OSPF, meaning every packet will be authenticated. Obviously, you need the same password on both routers in order to make things work.
* Stub area flag: Besides area numbers, OSPF has different area types. Both routers have to agree on the area type to become neighbors.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-loopback-interface.png" alt=""><figcaption></figcaption></figure>

Each OSPF router needs to have a unique router ID which is based on the highest IP address on any active interface. There is a catch here, however. On any Cisco router, you can create loopback interfaces which are like a virtual interfaces. You can configure an IP address on it, and when you try to ping it, you will always get a response.

If you have a loopback interface on your OSPF router, then this IP address will be used as the router ID even when it’s not the highest active IP address. Why does OSPF do this? Well, it makes sense…your loopback interface will never go down unless your router crashes.

Using a loopback interface, you can do two things:

* Advertise the IP address on the loopback interface in OSPF.
* Don’t advertise the IP address on the loopback interface in OSPF.

What’s the difference? If you advertise a loopback interface, other routers will be able to reach and ping the IP address on this loopback interface or even use it to telnet into the router. If you don’t, then well, you can’t…it’s as easy as that.

Everything is well. We have configured OSPF…we have become neighbors with a bunch of routers and they have exchanged link-state advertisements. Our routers have built their LSDB and they have a full topology picture of our network. Next step is to run the SPF algorithm and see what the shortest path to our destination is.

Remember metrics? The metric is what the routing protocol uses in order to determine the best path. OSPF uses a metric called cost which is based on the bandwidth of an interface, it works like this:

> Cost = Reference bandwidth / Interface bandwidth.

The reference bandwidth is a default value on Cisco routers which is a 100Mbit interface. You divide the reference bandwidth by the bandwidth of the interface and you’ll get the cost.

Example: If you have a 100 Mbit interface what will the cost be?

Cost = Reference bandwidth / Interface bandwidth.

100 Mbit / 100 Mbit = COST 1

Example: If you have a 10 Mbit interface what will the cost be?

100 Mbit / 10 Mbit = COST 10

Example: If you have a 1 Mbit interface what will the cost be?

100 Mbit / 1 Mbit = COST 100

The lower the cost the better the path is.

Look at the picture below, we are looking at R1 and running the SPF algorithm looking for the shortest path to our destination. Which one are we going to use?

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-five-routers-destination.png" alt=""><figcaption></figcaption></figure>

Using R2 we would have a cost of 10+8 +5 which is 23. The path in the middle through R3 is 5+5+5 = 15. The path through R3 has a cost of 20+20+5 = 45. The path in the middle through R3 has the lowest cost so this is the path we are going to use!

Let’s look at another scenario:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-five-routers-destination-ecmp.png" alt=""><figcaption></figcaption></figure>

As you can see the path through the R2 and R3 have the same cost (5+5+5 = 15). What are we going to do here? The path is equal.

The answer is load balancing. We are going to use both paths and OSPF will load balance among them 50/50. Some things worth knowing about OSPF load balancing:

* Paths must have an equal cost.
* OSPF adds paths with an equal cost in the routing table.
* The default value is a maximum of 4 equal-cost paths.
* The maximum value is 32 equal-cost paths (this could depend on your platform and/or IOS version, though)
* To make paths equal cost, change the “cost” of a link.

If a path is not equal, we can make it so by manually changing the cost or bandwidth of an interface.

{% hint style="info" %}
IOS 15 supports a maximum of 32 paths for OSPF.
{% endhint %}

One last thing I’d like to tell you about OSPF is authentication:

* OSPF can do MD5 authentication.
* OSPF can do clear text authentication.
* You can enable authentication for the entire area or a single interface.

That’s all I wanted to explain to you about OSPF for now. I hope this has been useful to you and that you now understand how OSPF works. In another lesson, I will demonstrate how to configure it. If you enjoyed it, please leave a comment!
