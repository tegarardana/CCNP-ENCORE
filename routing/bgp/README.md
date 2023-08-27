# BGP

## Introduction to BGP

This lesson will be interesting! BGP (Border Gateway Protocol) is the routing protocol that glues the Internet together. I’m going to explain in which situations we need BGP and how it works.

Before you continue reading I should tell you to “forget” everything you know about routing protocols like [RIP](https://networklessons.com/rip/how-to-configure-rip-on-a-cisco-router), [OSPF](https://networklessons.com/cisco/ccnp-encor-350-401/introduction-to-ospf) and [EIGRP](https://networklessons.com/cisco/ccnp-encor-350-401/introduction-to-eigrp) so far…Those three routing protocols have one thing in common: they are all IGPs (Interior Gateway Protocols). We only use them within our autonomous system but they are not scalable to use for a network as large as the Internet.

RIP, OSPF and EIGRP are all different but they have one thing in common…they want to find the shortest path to the destination. When we look at the Internet we don’t care as much as to find the shortest path, being able to manipulate traffic paths is far more important. There is only one routing protocol we currently use on the Internet which is BGP.

### Why do we need BGP?

Let’s start by looking at some scenarios so you can understand why and when we need BGP:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/isp-customer-internet.png" alt=""><figcaption></figcaption></figure>

Nowadays almost everything is connected to the Internet. In the picture above we have a customer network connected to an ISP (Internet Service Provider). Our ISP is making sure we have Internet access. Our ISP has given us a **single public IP address** we can use to access the Internet. To make sure everyone on our LAN at the customer side can access the Internet we are using **NAT/PAT (Network / Port address translation)** to translate our internal private IP addresses to this single public IP address. This scenario is excellent when you only have clients that need Internet access. On our customer LAN we only need a default route pointing to the ISP router and we are done. For this scenario we don’t need BGP…

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/isp-customer-servers-internet.png" alt=""><figcaption></figcaption></figure>

Maybe the customer has a couple of servers that need to be reachable from the Internet…perhaps a mail- or webserver. We could use port forwarding and forward the correct ports to these servers so we still only need a single IP address. Another option would be to get more public IP addresses from our ISP and use these to configure the different servers. For this scenario we still don’t need BGP…

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/isp-customer-servers-internet-redundancy.png" alt=""><figcaption></figcaption></figure>

What if I want a bit more redundancy? Having a single point of failure isn’t a good idea. We could add another router at the customer side and connect it to the ISP. You can use the primary link for all traffic and have another link as the backup. We still don’t require BGP in this situation, it can be solved with default routing:

* Advertise a default route in your IGP on the primary customer router with a low metric.
* Advertise a default route in your IGP on the secondary customer router with a high metric.

This will make sure that your IGP sends all traffic using the primary link. Once the link fails your IGP will make sure all traffic is sent down the backup link. Let me ask you something to think about…can we do any load balancing across those two links? It’ll be difficult right?

Your IGP will send all traffic down the primary link and nothing down the backup link unless there is a failure. You could advertise a default route with the same metric but you’d still have something like a 50/50% load share. What if I wanted to send 80% of the outgoing traffic on the primary link and 20% down the backup link? That’s not going to happen here but with BGP it’s possible.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/customer-two-isps-bgp.png" alt=""><figcaption></figcaption></figure>

This scenario is a bit more interesting. Instead of being connected to a single ISP we now have two different ISPs. For redundancy reasons it’s important to have two different ISPs, in case one fails you will always have a backup ISP to use. What about our Customer network? We still have two servers that need to be reachable from the Internet.

In my previous examples we got public IP addresses from our ISP. Now I’m connected to two different ISPs so what public IP addresses should I use? From ISP1 or ISP2? If we use public IP addresses from ISP1 (or ISP2) then these servers will be unreachable once the ISP has connectivity issues.

Instead of using public IP addresses from the ISP we will get our own public IP addresses.The IP address space is maintained by IANA (Internet Assigned Numbers Authority – [http://www.iana.org/](http://www.iana.org/) ). IANA is assigning IP address space to a number of large Regional Internet Registries like [RIPE ](http://www.ripe.net/)or [ARIN](https://www.arin.net/). Each of these assign IP address space to ISPs or large organizations.\
When we receive our public IP address space then we will advertise this to our ISPs. Advertising is done with a routing protocol and that will be BGP.

If you are interested here’s an overview of the IPv4 space that has been allocated by IANA:

[IANA IPv4 address space](http://www.iana.org/assignments/ipv4-address-space/ipv4-address-space.xml)

### Autonomous Systems

Besides getting public IP address space we also have to think about an **AS (Autonomous System)**:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/autonomous-system-numbers.png" alt=""><figcaption></figcaption></figure>

An AS is a collection of networks under a single administrative domain. The Internet is nothing more but a bunch of autonomous systems that are connected to each other. Within an autonomous system we use an IGP like OSPF or EIGRP.

For routing between the different autonomous systems we use an **EGP (external gateway protocol).** The only EGP we use nowadays is BGP.

How do we get an autonomous system number? Just like public IP address space you’ll need to register one.

Autonomous system numbers are 16-bit which means we have 65535 numbers to choose from. Just like private and public IP addresses, we have a range of public and private AS numbers.

Range 1 – 64511 are globally unique AS numbers and range 64512 – 65535 are private autonomous system numbers.

If you are interested, see if you can find the AS number of your ISP:

[UltraTools AS Information Lookup](https://www.ultratools.com/tools/asnInfo)

BGP has two flavors:

* **External BGP**: used between autonomous systems
* **Internal BGP**: used within the autonomous system.

External BGP is to exchange routing information between the different autonomous systems. In [this lesson](https://networklessons.com/cisco/ccnp-encor-350-401/internal-bgp-border-gateway-protocol-explained) I explain why we need internal BGP. I would recommend to read it after finishing this lesson and learning about [external BGP](https://networklessons.com/cisco/ccnp-encor-350-401/how-to-configure-ebgp-external-bgp) first.

### BGP Advertisements

You now have an idea of why we require BGP and what autonomous systems are. The Internet is a big place, as I am writing this there are more than 500.000 prefixes in a complete Internet routing table. If you are curious, you can find the size of the Internet routing table here:

[CIDR Report](http://www.cidr-report.org/as2.0/)

On the internet there are a number of looking glass servers. These are routers that have public view access and you can use them to look at the Internet routing table. If you want to see what it looks like check out:

[Looking glass servers](http://www.bgp4.as/looking-glasses)

Scroll down all the way to “Category 2 – IPv4 and IPv6 BGP Route Servers by region (TELNET access)”. You can telnet to these devices and use **show ip route** and **show ip bgp** to check the BGP or routing table.

When we run BGP, does this mean we have to learn more than 500.000 prefixes? It depends…let’s look at some examples:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-as1-as2-as3-isp-customer.png" alt=""><figcaption></figcaption></figure>

Above in our picture our customer network has an autonomous system number (AS 1) and some IP address space (10.0.0.0 /8), let’s pretend that these are public IP addresses. We are connected to two different ISPs and you can see their AS number (AS2 and AS3) and IP address space (20.0.0.0/8 and 30.0.0.0/8). We can reach the rest of the internet through both ISPs.

We can use BGP to advertise our address space to the ISPs but what are the ISPS going to advertise to our customer through BGP? There are a number of options:

* They advertise only a default route.
* They advertise a default route and a partial routing table.
* They advertise the full Internet routing table.

Let’s walk through these three options!

#### Default Route

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/isp-bgp-default-route.png" alt=""><figcaption></figcaption></figure>

Receiving a default route requires the **fewest resources** on your routers since you only have a single entry to reach any external network. The customer router will advertise its 10.0.0.0 /8 network to both ISPs which will advertise it to any other AS they are connected to and we will use a default route to reach anything on the Internet. The downside of this configuration is that our customer network doesn’t know what is behind ISP1 and ISP2. We have connectivity because of the default routes but this can lead to sub-optimal routing. If we only have the default routes then we will send all traffic to one of the ISPs.

Here’s what could happen if you only use default routes:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-default-route-sub-optimal-routing.png" alt=""><figcaption></figcaption></figure>

Our customer network only received a default route from both ISPs and we have chosen to use the default route of ISP1 to send all our outgoing traffic to. This means that whenever we send traffic meant for 30.0.0.0 /8 (ISP2) it’s going to be sent to ISP1 and then to ISP2. It’s not a problem but it’s not optimal.

#### Partial Routing Updates

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-isp-partial-routing.png" alt=""><figcaption></figcaption></figure>

We can also receive a **partial routing table** plus a default route. This partial update might include all the IP address space that the ISPs have assigned to their customers.

Just like in real life…the more you know the better off you are (unless you believe ignorance is bliss). In the world of routing having more routing information means you can make better routing decisions. We’ll have less sub-optimal routing problems than when we only have the default route.

#### Full Internet Routing Table

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-full-internet-routing-table.png" alt=""><figcaption></figcaption></figure>

The last option that we have is that we receive the full Internet routing table from both ISPs. This requires more resources but we’ll be able to make the best routing decisions.

### Path Vector

BGP is called a path vector routing protocol. What does this mean? Take a look at this image:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-as1-as2-as3-as4.png" alt=""><figcaption></figcaption></figure>

We have 4 autonomous systems and we are running BGP to exchange routing information. In AS 1 we have network 1.1.1.0 /24 and this is advertised to AS 2, AS 3 and AS 4.

If we would look at the BGP table of the router in AS4 then we will see network 1.1.1.0 /24 but it also stores the **path** we have to get through in order to get there. It will store the prefix but also the paths it has to cross in order to get to 1.1.1.0 /24. Here’s an example of a real BGP router:

<pre><code><strong>route-views.optus.net.au>show ip bgp
</strong>BGP table version is 128380331, local router ID is 203.202.125.6
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  1.0.0.0/24       202.160.242.71                         0 7473 15169 i
</code></pre>

The output above is from one of the BGP looking glass servers.

By using the **show ip bgp** command I can look at the BGP table and we see this router knows about network 1.0.0.0 /24. The next-hop IP address is 202.160.242.71. At the end of the line you see path with the numbers 7473 15169. These are the autonomous systems we have to get through in order to get to this network.

### BGP Route Selection

What all IGPs have in common is that all of them want to find the shortest path to the destination. BGP works differently, since autonomous systems belong to different ISPs or organizations we want to be able to selectively influence our routing. Take a look at this example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-9-autonomous-systems.png" alt=""><figcaption></figcaption></figure>

BGP allows us to use routing policies at the autonomous system level. In the picture above I have 9 autonomous systems and in AS 9 we have network 192.168.9.0 /24. If we look at AS 1 then we have a lot of different paths we can take to reach network 192.168.9.0 /24 in AS 9.

Does this mean the network administrator at AS 1 can choose the path we are going to use? Not really because of the following reasons:

* You can choose the **exit path**…AS1 can send traffic to AS 2 or AS4 but you don’t make routing decisions for other autonomous systems.
* Each autonomous system will only advertise the **best path** to your autonomous system. AS 1 will only learn about the **best path** from AS 2 and AS 4 unless their best path fails…only then you will learn about the second best path.

BGP uses a set of **BGP attributes** to select a path, these are covered in other lessons.

### Conclusion

Hopefully this lesson has been helpful to understand the basics of BGP and why we use it. In other lessons we will take a closer look at the configuration of external and internal BGP and also how the BGP path selection works.

If you have any questions, feel free to leave a comment!
