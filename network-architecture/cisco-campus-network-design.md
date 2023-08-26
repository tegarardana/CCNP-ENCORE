# Cisco Campus Network Design

In this lesson we’ll take a look at some of the basics of Cisco Campus network design.

What is a “campus” network anyway?

A campus network is an enterprise network (hundreds or thousands of users) where we have one or more LANs in one or multiple buildings. Everything is geographically close to each other so we typically use Ethernet (and Wireless) for connectivity. Typically the company owns everything on the campus…hardware, cabling, etc.

To support this many users we require a lot of switchports which means a lot of switches. We need a physical design to connect these switches to each other and also a good logical design to make it work.

Let’s take a look at some networks to see how they “grow” and some design issues that we will face. Let’s start with a simple example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/computers-connected-to-hub.png" alt=""><figcaption></figcaption></figure>

Back in the old days we used to have hubs so we had half-duplex networks. When one host would transmit something, the others had to wait. When two hosts would send at the same time we’d get a collision and we used the CSMA/CD algorithm to deal with these collisions. Everything that is connected to the hub is a **single collision domain**. Also, whenever a host sends a broadcast everyone will receive it. There’s only one **broadcast domain**.

In this example there are only 5 hosts so it’s no problem but when you have hundreds of hosts the collisions and broadcasts will have a serious impact on the available bandwidth. To reduce the size of the collision domain we started using bridges and then switches. The broadcast domains can be reduced by using VLANs. Here’s an example:

\
Now we have a single switch and some hosts that are in different VLANs. Each port on the switch is a collision domain and each VLAN is a separate broadcast domain. If we use a multilayer switch, the VLANs will be able to communicate with each other.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/computers-vlan-10-20-switch-1.png" alt=""><figcaption></figcaption></figure>

Once this network grows we might not have enough switchports anymore on a single switch. You could add a second switch and connect it to the first one but what if we add a third of fourth switch? How are we going to connect them to each other?

If you don’t think about your design beforehand, you might end up with something like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/bad-switch-network-design.png" alt=""><figcaption></figcaption></figure>

Switches, hosts, cables and VLANs everywhere. Before we know it, the network is one big spaghetti.

We need a network that is easy to maintain, offers high availability, scalability and is able to quickly respond to changes in the topology. To achieve all of this, Cisco has a **hierarchical approach to network design** where we have **multiple layers** in the network. Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/hierarchical-switch-network-design.png" alt=""><figcaption></figcaption></figure>

In this design we have an access layer and distribution layer. The access layer is close to the end users, these are switches that we use to connect computers, laptops, access points and more. The distribution layer is used to aggregate all the different access layer switches.

The advantage of this hierarchical network design is that it’s scalable. When the campus grows and we get more users, building and floors then we can add multiple distribution layers. When this happens, we’ll add another layer:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/core-distribution-access-layers.png" alt=""><figcaption></figcaption></figure>

The core layer aggregates all the different distribution layer switches. This design also makes our traffic paths predictable and easy to visualize. Basically there are three different traffic flows:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/traffic-on-access-layer.png" alt=""><figcaption></figcaption></figure>

All traffic **starts at the access layer** and if needed it will move up the distribution and core layer. In this example the traffic is **local**; it doesn’t leave the access layer switch. This could be traffic between two hosts within the same VLAN. Here’s another example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/traffic-access-distribution-layer.png" alt=""><figcaption></figcaption></figure>

Traffic between hosts that are on different access layer switches has to cross the distribution layer switch. Finally, sometimes we have to cross the core layer:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/traffic-access-distribution-core-layer.png" alt=""><figcaption></figcaption></figure>

Each of the layers has a different function and requirements. Let’s discuss each layer:

Access Layer:

The main function of the access layer is to connect all end devices like computers, laptops, access points, IP phones, printers, etc.

* We require a lot of switchports to connect all these end devices. This means the price per switchport should be low, for this reason we typically use layer 2 switches on the access layer.
* Depending on our traffic flows, traffic from the access layer has to go towards the distribution layer so we might require multiple uplinks.
* POE (Power over Ethernet): if you have IP phones or wireless access points then we’ll need POE on the access layer.
* QoS features: If we use VoIP then we might need switches that support QoS to give precedence to VoIP traffic.
* Security: The access layer is the “entry” of our network. You might want to protect DHCP, ARP and spanning-tree from malicious devices that are connected to the access layer.

Distribution Layer:

The distribution layer connects the access and core layer together. Since this is where we aggregate the access layer, we need sufficient bandwidth up to the core layer. Typically the distribution layer is where we use routing, this is where we terminate the VLANs from the access layer.

* We use routing on the distribution layer so we need switches that are capable of high throughput routing performance.
* Multiple redundant uplinks to the access and core layer. If a distribution layer switch fails, multiple access layer switches might lose connectivity.
* QoS features: just like the access layer, we might need QoS to give preference to certain traffic like VoIP.
* Security: we use access-lists on the distribution layer to filter certain inter-VLAN traffic.

Core Layer:

The core aggregates all the different distribution layer switches, this is the backbone of our network. This means the switches in the core layer should be able to handle all traffic from the distribution layer switches. Also, if the core fails all connectivity between distribution layers will be impossible.

* High bandwidth / throughput required.
* High availability / redundancy required. Think about multiple links, redundant power supplies, and redundant supervisors (CPU).
* QoS features: Qos is end-to-end in the campus so we also need support in the core.
* No packet manipulation: We don’t configure access-lists or make changes to packets in the core.

Sometimes the size of the network is too small to justify a separate core layer. In this case we the function of the core and distribution layer is combined into a single layer. This is called the **collapsed core**.

The three layer model is pretty straight forward but we haven’t talked about real redundancy yet. In all of the models I showed you so far we only had one link between the switches. Let’s look at redundancy between the access and distribution layer first:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/redundancy-access-distribution-layer.png" alt=""><figcaption></figcaption></figure>

In this example we added some redundancy. Instead of a single distribution layer switch we now have two switches and each access layer switch is connected to both distribution layer switches. The link in between the distribution layer switches is required so that they can reach each other directly (needed for (routing) protocols).

We still have one problem…there’s no redundancy in the core yet. Let’s add it:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/hierarchical-network-redundant-links.png" alt=""><figcaption></figcaption></figure>

The core is the backbone and the most important part of the network so we need redundancy here. We’ll add another switch and redundant links to our distribution layer switches.

The design above is still pretty simple with only a few switches. What if we add more distribution layers and multiple access layers? Should we connect all distribution layer switches to the others? What about the access layer switches? Should we connect them to all distribution layer switches?

When the network grows, we create so-called **switch blocks**. A switch block are two distribution layer switches with access layer switches beneath them. Each switch block is then connected to the core layer:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/hierarchical-network-switch-blocks.png" alt=""><figcaption></figcaption></figure>

This is a nice and tidy hierarchical network diagram. You can see two switch blocks, each switch block has two distribution layer switches and there are multiple access layer switches. There is no connectivity between the different switch blocks, only with the core layer. Switch blocks are also used to connect (large) server farms to the core layer.

The size of a switch block really depends on the number of users and the applications / traffic types. Network analysis is required to see what traffic patterns there are on the network and what kind of bandwidth requirements we have.

So far we mostly talked about physical topologies, what about logical topologies? I explained earlier that we typically use layer 2 on the access layer and layer 3 (routing) on the distribution layer but there’s more to this story. Let’s look at some examples:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/distribution-l3-access-l2.png" alt=""><figcaption></figcaption></figure>

In this example we have a switch block with two distribution layer switches and two access layer switches. There’s one VLAN that is used on both access layer switches.

Between the distribution and access layer switches we use layer 2, from the distribution to the core layer we use layer 3. The distribution layer switches are used as the routers for the devices behind the access layer switches.

Because VLAN 10 is used on both access layer switches, the link between the two distribution layer switches HAS to be a layer 2 link as well. There are two reasons for this:

* If one of the uplinks from the access to the distribution layer fails, VLAN 10 traffic could go through the access layer.
* The switches on the distribution layer will use a protocol to create a virtual gateway IP address. We need layer two connectivity for this and you don’t want this traffic to go through the access layer.

The dashed lines indicate where we need VLAN 10 which is on every link…this will work but it’s not optimal. We will require spanning-tree to create a loop-free topology and the entire switch block becomes a single point of failure.

If a host starts sending a lot of broadcast frames then the entire switch block is affected. A better solution would be this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/vlan-per-access-layer-switch.png" alt=""><figcaption></figcaption></figure>

Each access layer switch now has a single VLAN, this has a number of advantages. First of all, there is no redundant topology within VLAN 10 or 20 now so we don’t have to rely on spanning-tree to create a loop-free topology…this makes the switch block more stable.

Also each VLAN can use both uplinks which allow load balancing. The link between the two distribution layer switches is now a layer 3 link. The third option is to use routing everywhere:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/routing-access-layer-switches.png" alt=""><figcaption></figcaption></figure>

We can push layer 3 even to the access layer if the switches support it. The advantage is that routing protocols typically have a fast(er) convergence time and you can use all available links. OSPF and EIGRP allow load balancing.

You have now seen the different layers and the options we have for layer 2 / 3 in the campus. Here are some of the key points that you should remember from this story:

* Each layer should have a pair of switches for redundancy.
* Each switch should be connected to the upper layer with two links for redundancy.
* Connect each pair of distribution layer switches together with a link, this is needed for some (routing) protocols.
* Don’t connect the access layer switches to each other.
* Don’t extend VLANs above the distribution layer, this is our boundary. From the distribution layer up to the core layer we only use routing.
* If possible, don’t span VLANs over multiple access layer switches.
* The core layer is simple…we don’t do any packet manipulation but we require high bandwidth, availability and scalability since it aggregates all the distribution layers.

This should give you an idea about the different layers and what a (large) campus network design looks like. What about the switches? What models should we use in each layer? Here’s an overview with switches that are commonly used in the access layer:

<table><thead><tr><th width="73">Model</th><th width="162">Port Density</th><th width="78">Uplinks</th><th width="90">Backplane</th><th width="235">Features</th></tr></thead><tbody><tr><td> </td><td></td><td></td><td></td><td></td></tr><tr><td><strong>2960-X</strong></td><td>384 (8x 48-port switches stacked)</td><td>2x 10GE or 4x GE</td><td>80 Gbps</td><td>RIP, OSPF, POE+</td></tr><tr><td><strong>3650</strong></td><td>432 (9x 48-port switches stacked)</td><td>4x 10GE or 2x GE</td><td>160 Gbps</td><td>All routing protocols, POE+, wireless controller</td></tr><tr><td><strong>3850</strong></td><td>432 (9x 48-port switches stacked)</td><td>4x 10GE or 4x GE</td><td>480 Gbps</td><td>All routing protocols, POE+, wireless controller integrated</td></tr><tr><td><strong>4500E</strong></td><td>384 (8x 48-port modules in chassis)</td><td>Up to 12x 10GE per module</td><td>928 Gbps</td><td>Dual supervisors, all routing protocols, integrated wireless controller.</td></tr></tbody></table>

And here are some commonly used switches used in the distribution and/or core layer:

<table><thead><tr><th width="73">Model</th><th width="162">Port Density</th><th width="90">Backplane</th><th width="312">Features</th></tr></thead><tbody><tr><td> </td><td></td><td></td><td></td></tr><tr><td><strong>4500-X</strong></td><td>80x 10GE</td><td>1.6 Tbps</td><td>Dual chassis VSS redundancy</td></tr><tr><td><strong>4500-E</strong></td><td>96x 10GE or 384x GE</td><td>928 Gbps</td><td>Dual supervisors</td></tr><tr><td><strong>6807-XL</strong></td><td>40x 40Gbps, 160x GE and 480x GE</td><td>22.8 Tbps</td><td>Dual supervisors, dual chassis VSS redundancy</td></tr></tbody></table>

If you want to see the actual specifications then I would recommend to take a look at the Cisco website. That’s all we have for now, I hope you learned some of the basics of Cisco campus network design and the different building blocks that we use. If you have any questions, feel free to leave a comment.
