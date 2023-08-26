# Spine and Leaf Topology

We used Cisco’s three-layer hierarchical architecture for more than a decade, but in data centers, the spine-leaf architecture is more popular nowadays. In this lesson, you will learn about the spine-leaf architecture and its advantages.

Before we do that, you need to understand how data centers evolved and the disadvantages of the three-layer hierarchical architecture.

## Data Center Evolution

Let’s start with a short history lesson of how the three-layer architecture evolved throughout the years.

### Three-layer hierarchical architecture

Here’s an overview of the three-layer model:

![Three Layer Architecture Model Cisco](https://cdn.networklessons.com/wp-content/uploads/2019/10/three-layer-architecture-model-cisco.png)

For over a decade, we used this architecture with the three layers:

* Core
* Distribution (aggregation)
* Access

The access layer is where we connect our end devices. In a campus network, these are usually computers, laptops, and access points. In a data center, this is where our servers are. The distribution layer has redundant connections to access layer switches and connects to the core layer. The core layer provides fast transport between distribution layer switches.

Between the access layer and the distribution layer, we use L2 and spanning-tree (STP) to **block all links except one**. Between the distribution and core layer, we use routing.

For a detailed explanation, you can take a look at the [campus network design lesson](https://networklessons.com/cisco/ccnp-encor-350-401/cisco-campus-network-design-basics).

### vPC

To overcome the limitations of STP, Cisco introduced virtual-port-channels (vPC) in 2010. **vPCs offer active-active uplinks** from the access layer switches to the distribution layer switches. vPCs allow us to use all available bandwidth.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/10/three-layer-architecture-model-cisco-vpc.png" alt=""><figcaption></figcaption></figure>

### L2 Topology

Because of [virtualization](https://networklessons.com/cisco/ccnp-encor-350-401/introduction-to-cloud-computing), we can now pool the computing, networking, and storage resources in a pod into virtual resources. Pooling these resources often requires large L2 domains that span from the access layer up to the core layer.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/10/three-layer-architecture-model-l2-domain.png" alt=""><figcaption></figcaption></figure>

With these large L2 domains that span across the entire network, we can create a flexible resource pool and reallocate resources where needed.

An example is our servers. Before virtualization, we had physical servers in a single pod. With virtualization, we have hypervisors in multiple pods. A virtual machine that runs on a hypervisor in pod one can move to a hypervisor in pod two without any downtime. VMWare’s VMotion can do this but requires L2 connectivity to do it.

A side effect of having resources spread out over the network instead of within a single pod is that the **amount of east-west traffic increases**.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/10/three-layer-architecture-model-east-west-traffic.png" alt=""><figcaption></figcaption></figure>

Traffic between the two servers has to go through all layers of our network.

### L3 Topology

Another option is to use routing everywhere. The advantages of a routed topology are that we can use all links for forwarding and routing protocols converge faster than STP. If we require L2 connectivity between servers in different pods, we can use a VXLAN overlay network if needed.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/10/three-layer-architecture-model-l3-topology.png" alt=""><figcaption></figcaption></figure>

### Advantages and disadvantages

The three-layer hierarchical architecture has some advantages and disadvantages. Let’s take a look.

#### **Advantages**

This model offers the following advantages:

* **Availability**: When a pod goes down, the issue is usually isolated to one pod and doesn’t affect other pods.
* **Security**: We use L2 between the access and distribution layers and L3 between the distribution layers and core layer. We can filter traffic on L3 so we can decide what traffic goes into or outside of a pod.
* **Scalability**: When the network grows, we can easily add more distribution or access layer switches.
* **Familiarity**: We used this model for over a decade, so network engineers are familiar with this design and the protocols we use.

#### **Disadvantages**

There are, however, some disadvantages, which is why the spine-leaf architecture is gaining popularity.

Here are two important disadvantages:

* **Limited bandwidth**: vPCs solve the STP problem that we can only use one active link, but vPCs are limited to two active uplinks.
* **Latency**: The server-to-server latency could be high, depending on the traffic path. East-west traffic between pods has to go through the distribution and core layers.

## Spine and Leaf Architecture

The spine-leaf architecture was developed to overcome the limitations of the three-tier architecture. It offers high bandwidth, low latency, and non-blocking server-to-server connectivity **for data centers that primarily have east-west traffic flows**. Here’s what it looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/10/spine-leaf-architecture-model.png" alt=""><figcaption></figcaption></figure>

This topology is also known as a Clos architecture, created in the 1950s by Charles Clos to switch telephone calls. It’s a two-layer topology with spine and leaf switches.

Each leaf switch connects **to all spine switches**. The leaf switches connect to end devices like servers. The spine switches are the backbone. The traffic path from the leaf switches to the spine switches is random so that the **traffic load is distributed among all spine switches**. If a spine switch fails, there is only a slight performance degradation in the data center.

Expanding the capacity of a spine-leaf architecture is straightforward. When an uplink is oversubscribed, you can add a spine switch and connect all leaf switches to it. This **reduces the oversubscription ratio and increases the bandwidth** between the spine and leaf switches. When you need more ports for end devices, you add an extra leaf switch and connect it to all spine switches.

With the spine-leaf architecture, every end device is always a **maximum of two hops away**. There are only one spine switch and leaf switch between the source and destination. The only exception is when the destination is on the same leaf switch. This keeps latency and delays low and predictable.

A spine-leaf architecture can be L2 or L3. In an L2 design, we use Transparent Interconnection of Lots of Links (TRILL) or shortest path bridging (SPB) instead of STP.

### Advantages

The spine-leaf architecture has several advantages:

* **Improved redundancy**: Every leaf switch connects to every spine switch. This is superior to the three-layer architecture where an access layer switch is only connected to two uplink distribution layer switches where we use STP to block one of the two links.
* **Increased bandwidth**: The ability to use multiple active links instead of one increases bandwidth. With STP, we only have one active link out of X possible links. TRILL and SPB can use multiple active links at the same time.
* **Improved scalability**: Each additional spine switch adds another path to get from one leaf switch to another, and because we use all links, it increases bandwidth. This makes it easy to scale out our network if needed.
* **Lower cost**: Fixed-configuration switches are switches with a fixed number of ports and are usually not expandable. With the three-tier architecture, we often require chassis switches to provide enough ports. With the spine-leaf architecture, spine switches only connect to leaf switches. There are no connections between spine switches. Many spine-leaf networks use fixed-configuration switches.
* **Lower latency and delay**: We always only have to travel through one spine switch and leaf switch to get to a destination (unless the destination is connected to the same leaf switch). This minimizes latency and ensures a consistent delay.
* **Power consumption**: Fixed-configuration switches require fewer watts per port.

### Disadvantages

Of course, this architecture also has some disadvantages.

* **Cabling requirement**: Because all leaf switches connect to all spine switches, you need a lot of cables.
* **A limited number of hosts**: The number of hosts we can support depends on the number of ports on your spine and leaf switches. If you have spine switches with 32 ports, then you could connect 32 leaf switches.

## Conclusion

* We used the three-layer hierarchical architecture for over a decade. Nowadays, it’s no longer the best option for most datacenters.
* One of the issues is that STP blocks all links except one.
* vPC somehow solves this limitation of STP, but vPC can only use two active links.
* The amount of east-west traffic has increased; one of the reasons for this is virtualization.
* Some virtualization solutions require large L2 domains.
* East-west traffic has to go through the distribution layer, to the core layer, and down another distribution layer to get from one access layer to another. This increases latency and delay.
* The spine-leaf architecture has two layers: spine and leaf. It’s a good choice for data centers with primarily east-west traffic flows.
* Each leaf switch is connected to all spine switches.
* Every end device is only a maximum of two hops away. This reduces latency and delay.
* Spine-leaf topologies can be L2 or L3.
* In an L2 design, we use TRILL or SPB instead of STP. Both protocols can use all links to forward traffic.

I hope you enjoyed this lesson. If you have any questions, please leave a comment!
