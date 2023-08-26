# RSTP

Nowadays we see more and more routing in our networks. Routing protocols like OSPF and EIGRP are much faster than spanning-tree when they have to deal with changes in the network. To keep up with the speed of these routing protocols another flavor of spanning-tree was created…**rapid spanning-tree**.

Rapid spanning-tree is not a revolution of the original spanning-tree but an evolution. Behind the scenes some things have been changed to speed up the process, configuration-wise it’s the same as what you have seen so far. I will refer to the original spanning-tree as “classic spanning-tree”.

Let’s dive into rapid spanning-tree and we’ll see what the differences are with the classic spanning-tree. Take a look at the picture below:

<figure><img src="../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

Remember the port states of spanning-tree? We have a blocking, listening, learning and forwarding port state. This is the first difference between spanning-tree and rapid spanning-tree. Rapid spanning-tree only has three port states:

* **Discarding**
* Learning
* Forwarding

You already know about learning and forwarding but **discarding** is a new port state. Basically it combines the blocking and listening port state. Here’s a nice overview:

| **Classic Spanning-Tree** | **Rapid Spanning-Tree** | **Port active in topology?** | **Learns MAC addresses?** |
| ------------------------- | ----------------------- | ---------------------------- | ------------------------- |
| Blocking                  | Discarding              | No                           | No                        |
| Listening                 | Discarding              | Yes                          | No                        |
| Learning                  | Learning                | Yes                          | Yes                       |
| Forwarding                | Forwarding              | Yes                          | Yes                       |

Do you remember all the other port roles that spanning-tree has? Let’s do a little review and I’ll show you what is different for rapid spanning-tree:

<figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

The switch with the best bridge ID (priority + MAC address) becomes the root bridge. The other switches (non-root) have to find the shortest cost path to the root bridge. This is the root port. There’s nothing new here, this works exactly the same for rapid spanning-tree. The next step is to select the designated ports:

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

On each segment there can be only one designated port or we’ll end up with a loop. The port will become the designated port if it can send the best BPDU. SW1 as a root bridge will always have the best ports so all of interfaces will be designated. The fa0/16 interface on SW2 will be the designated port in my example because it has a better bridge ID than SW3. There’s still nothing new here compared to the classic spanning-tree. The interfaces that are left will be blocked:

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

SW3 receives better BPDUs on its fa0/16 interface from SW2 and thus it will be blocked. This is the alternate port and it’s still the same thing for rapid spanning-tree. Let me show you a new example with a port state that is new for rapid spanning-tree:

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

Here is a new port for you, take a look at the fa0/17 interface of SW2. It’s called a **backup port** and it’s new for rapid spanning-tree. You are very unlikely to see this port on a production network though. Between SW2 and SW3 I’ve added a hub. Normally (without the hub in between) both fa0/16 and fa0/17 would be designated ports.

Because of the hub the fa0/16 and fa0/17 interface on SW2 are now in the **same collision domain.** Fa0/16 will be elected as the designated port and fa0/17 will become the **backup port** for the fa0/16 interface. The reason that SW2 sees the fa0/17 interface as a backup port is because it receives its own BPDUs on the fa0/16 and fa0/17 interfaces and understands that it has two connections to the same segment. If you remove the hub the fa0/16 and fa0/17 will both be designated ports just like the classic spanning-tree.

Something else that is different is the BPDU, take a look:

<figure><img src="../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

The BPDU is different for rapid spanning-tree. In the classic spanning-tree the **flags** field only had two bits in use:

* Topology change.
* Topology change acknowledgment.

All bits of the flag field are now used. The role of the port that originates the BPDU will be added by using the **port role** field, it has the following options:

* Unknown
* Alternate / Backup port.
* Root port.
* Designated port.

This new BPDU is called a **version 2 BPDU.** Switches running the old version of spanning-tree will drop this new BPDU version. In case you are wondering…rapid spanning-tree and the old spanning **are compatible!** Rapid spanning-tree has a way of dealing with switches running the older spanning-tree version.

Let’s walk through the other stuff that has been changed:

BPDUs are now sent **every hello time**. Only the root bridge generated BPDUs in the classic spanning-tree and those were relayed by the non-root switches if they received it on their root port. Rapid spanning-tree works differently…all switches generate BPDUs **every two seconds (hello time).** This is the default hello time but you can change it.

The classic spanning-tree uses a max age timer (20 seconds) for BPDUs before they are discarded. Rapid spanning-tree works differently! BPDUs are now used as a **keepalive mechanism** similar to what routing protocols like OSPF or EIGRP use. If a switch **misses** **three BPDUs** from a neighbor switch it will assume connectivity to this switch has been lost and it will remove all MAC addresses immediately.

Rapid spanning tree will **accept inferior BPDUs**. The classic spanning tree ignores them. Does this ring a bell? This is pretty much the backbone fast feature of classic spanning-tree.

Transition speed (convergence time) is the most important feature of rapid spanning tree. The classic spanning tree had to walk through the listening and learning state before it would move an interface to the forwarding state, this took 30 seconds with the default timers. The classic spanning tree was based on **timers**.

Rapid spanning **doesn’t use timers** to decide whether an interface can move to the forwarding state or not. It will use a **negotiation mechanism** for this. I’ll show you how this works in a bit.

Do you remember portfast? If we enable portfast while running the classic spanning tree it will skip the listening and learning state and put the interface in forwarding state right away. Besides moving the interface to the forwarding state it will also **not generate topology changes** when the interface goes up or down. We still use portfast for rapid spanning tree but it’s now referred to as an **edge port**.

Rapid spanning tree can only put interfaces in the forwarding state really fast on **edge ports (portfast)** or **point-to-point interfaces**. It will take a look at the **link type** and there are only two link types:

* Point-to-point (full duplex)
* Shared (half duplex)

Normally we are using switches and all our interfaces are configured as full duplex, rapid spanning tree sees these interfaces as point-to-point. If we introduce a hub to our network we’ll have half duplex which is seen as a shared interface to rapid spanning-tree.

Let’s take a close look at the negotiation mechanism that I described earlier:

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

Let me describe the rapid spanning tree synchronization mechanism by using the picture above. SW1 on top is the root bridge. SW2, SW3 and SW4 are non-root bridges.

As soon as the link between SW1 and SW2 comes up their interfaces will be in blocking mode. SW2 will receive a BPDU from SW1 and now a **negotiation** will take place called **sync:**

<figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

After SW2 received the BPDU from the root bridge it **immediately blocks all its non-edge designated ports**. Non-edge ports are the interfaces that connect to other switches while edge ports are the interfaces that have portfast configured. As soon as SW2 blocks its non-edge ports the link between SW1 and SW2 will go into forwarding state. SW2 will now do the following:

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

SW2 will also perform a sync operation with both SW3 and SW4 so they can quickly move to the forwarding state.

Are you following me so far? The lesson to learn here is that rapid spanning tree uses this **sync mechanism instead of the “timer-based” mechanism** that the classic spanning tree uses (listening > learning > forwarding). I’m going to show you what this looks like on real switches in a bit. Let’s take a closer look at the sync mechanism, let’s look at what happens exactly between SW1 and SW2:

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

At first the interfaces will be blocked until they receive a BPDU from each other. At this moment SW2 will figure out that SW1 is the root bridge because it has the best BPDU information. The sync mechanism will start because SW1 will set the **proposal bit** in the flag field of the BPDU. When SW2 receives the proposal it has to do something with it:

<figure><img src="../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

SW2 will block all its non-edge interfaces and will start the synchronization towards SW3 and SW4, once this is done SW2 will let SW1 know about this:

<figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

Once SW2 has its interfaces in sync mode it will let SW1 know about this by sending an **agreement.** This agreement is a **copy of the proposal BPDU** where the proposal bit has been switched off and the agreement bit is switched on. The fa0/14 interface on SW2 will now go into forwarding mode. When SW1 receives the agreement here’s what happens:

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

Once SW1 receives the agreement from SW2 it will put its fa0/14 interface in forwarding mode immediately.

What about the fa0/16 and fa0/19 interface on SW2?

<figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

The exact same sync mechanism will take place now on these interfaces. SW2 will send a proposal on its fa0/16 and fa0/19 interfaces towards SW3 and SW4. SW3 and SW4 will send an agreement:

<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

SW3 and SW4 don’t have any other interfaces so they will send an agreement back to SW2:

<figure><img src="../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

SW2 will place its fa0/16 and fa0/19 interface in forwarding and we are done. This sync mechanism is just a couple of messages flying back and forth and very fast, it’s much faster than the timer-based mechanism of the classic spanning tree!

What else is new with rapid spanning tree? There are three more things I want to show you:

* UplinkFast
* Topology change mechanism.
* Compatibility with classic spanning tree.

When you configure the classic spanning tree you have to enable UplinkFast yourself. Rapid spanning tree uses UpLinkFast by default, you don’t have to configure it yourself. When a switch loses its root port it will put its alternate port in forwarding immediately.

The difference is that the classic spanning tree needed multicast frames to update the MAC address tables of all switches.\


**We don’t need this anymore** because the topology change mechanism for rapid spanning tree is different. So what’s different about the topology change mechanism?

With the classic spanning tree a link failure would trigger a topology change. Using rapid spanning tree a **link failure is not considered as a topology change**. Only non-edge interfaces (leading to other switches) that move to the forwarding state are considered as a topology change. Once a switch detects a topology change this will happen:

* It will start a **topology** **change while timer** with a value that is twice the hello time. This will be done for all non-edge designated and root ports.
* It will **flush the MAC addresses** that are learned on these ports.
* As long as the topology change while timer is active it will set the topology change bit on BPDUs that are sent out these ports. BPDUs will also be sent out of its root port.

When a neighbor switch receives this BPDU with the topology change bit set this will happen:

* It will clear all its MAC addresses on all interfaces except the one where it received the BPDU with the topology change on.
* It will start a topology change while timer itself and send BPDUs on all designated ports and the root port, setting the topology change bit.

<figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

Instead of sending a topology change all the way up to the root bridge like the classic spanning tree does, the topology change is now quickly flooded throughout the network. If you want to know exactly how this topology change mechanism works then take a look at [this lesson](https://networklessons.com/cisco/ccnp-encor-350-401/spanning-tree-topology-change-notification-tcn).

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

Last but not least, let’s talk about compatibility. The short answer is that rapid spanning tree and classic spanning tree **are** **compatible.** However when a switch running rapid spanning tree communicates with a switch running classic spanning tree all the Speedy Gonzales features won’t work!

In the example above I have my three switches. Between SW1 and SW2 we will run rapid spanning tree. Between SW2 and SW3 we will fall back to the classic spanning tree.

Seen enough theory? In the [next lesson](https://networklessons.com/cisco/ccnp-encor-350-401/rapid-spanning-tree-configuration) I will show you the [configuration and the debugs](https://networklessons.com/cisco/ccnp-encor-350-401/rapid-spanning-tree-configuration) of everything that you have learned so far.
