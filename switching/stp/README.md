# STP

## Introduction to Spanning-Tree

Spanning-tree is a protocol that runs on our switches that helps us to solve loops. Spanning-tree is one of the protocols that you must understand as a network engineer and you will encounter it for sure if you decide to face the Cisco CCNA R\&S exam. This lesson is an introduction to spanning-tree, you will learn why we need it, how it works and how you can check the spanning-tree topology on your Cisco switches.

### Why do we need spanning-tree?

What is a loop, and how do we get one? Let me show you an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/01/switches-single-cable-1.png" alt=""><figcaption></figcaption></figure>

In the picture above, we have two switches. These switches are connected with a single cable, so there is a **single point of failure**. To get rid of this single point of failure, we will add another cable:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/01/switches-redundant-cable-1.png" alt=""><figcaption></figcaption></figure>

With the extra cable, we now have **redundancy**. Unfortunately for us, redundancy also brings **loops**. Why do we have a loop in the scenario above? Let me describe it to you:

1. H1 sends an ARP request because it’s looking for the MAC address of H2. An ARP request is a **broadcast** frame.
2. SW1 will forward this broadcast frame on all it interfaces, except the interface where it received the frame on.
3. SW2 will receive both broadcast frames.

Now, what does SW2 do with those broadcast frames?

1. It will forward it from every interface except the interface where it received the frame.
2. This means that the frame that was received on interface Fa0/0 will be forwarded on Interface Fa1/0.
3. &#x20;The frame that was received on Interface Fa1/0 will be forwarded on Interface Fa0/0.

Do you see where this is going? We have a loop! Both switches will keep forwarding over and over again until the following happens:

* You fix the loop by disconnecting one of the cables.
* One of your switches will crash because they are overburdened with traffic.

Ethernet frames **don’t have a TTL** (Time to Live) value, so they will loop around forever. Besides ARP requests, many frames are broadcasted. For example, whenever the switch doesn’t know about a destination MAC address, it will be flooded.

### How spanning-tree solves loops

Spanning-tree will help us to create a **loop-free topology** by blocking certain interfaces. Let’s take a look at how spanning-tree work! Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/01/switches-in-triangle-1.png" alt=""><figcaption></figcaption></figure>

We have three switches, and as you can see, we have added redundancy by connecting the switches in a triangle, this also means we have a loop here. I have added the MAC addresses but simplified them for this example:

* SW1: MAC AAA
* SW2: MAC BBB
* SW3: MAC CCC

Since spanning-tree is enabled, all our switches will send a special frame to each other called a **BPDU (Bridge Protocol Data Unit)**. In this BPDU, there are two pieces of information that spanning-tree requires:

* **MAC address**
* **Priority**

The **MAC address** and the **priority** together make up the **bridge ID**. The BPDU is sent between switches as shown in the following picture:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/01/switches-send-bpdu-1.png" alt=""><figcaption></figcaption></figure>

Spanning-tree requires the bridge ID for its calculation. Let me explain how it works:

* First of all, spanning-tree will **elect a root bridge**; this root bridge will be the one that has the best “bridge ID”.
* The switch with the **lowest bridge ID** is the best one.
* By default, the priority is **32768,** but we can change this value if we want.

So who will become the root bridge? In our example, SW1 will become the root bridge! Priority and MAC address make up the bridge ID. Since the priority is the same on all switches, it will be the MAC address that is the tiebreaker. SW1 has the lowest MAC address thus the best bridge ID and will become the root bridge.

The ports on our root bridge are always **designated,** which means they are in a **forwarding** state. Take a look at the following picture:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/01/stp-root-elected-1.png" alt=""><figcaption></figcaption></figure>

Above, you see that SW1 has been elected as the root bridge and the “D” on the interfaces stands for designated.

Now we have agreed on the root bridge, our next step for all our **“non-root” bridges** (so that’s every switch that is not the root) will have to find the **shortest path to our root bridge**! The shortest path to the root bridge is called the **“root port”**. Take a look at my example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/01/stp-rootport-selected-1.png" alt=""><figcaption></figcaption></figure>

I’ve put an **“R” for “root port”** on SW2 and SW3. Their Fa0/0 interface is the shortest path to get to the root bridge. In my example, I’ve kept things simple, but “shortest path” in spanning-tree means it will actually look at the **speed of the interface.** Each interface has a certain cost, and the path with the lowest cost will be used. Here’s an overview of the interfaces and their cost:

* 10 Mbit = Cost 100
* 100 Mbit = Cost 19
* 1000 Mbit = Cost 4

Excellent!…we have designated ports on our root bridge and root ports on our non-root bridges, we still have a loop, however, so we need to shut down a port between SW2 and SW3 to break that loop. So which port are we going to shut down? The one on SW2 or the one on SW3? We’ll look again at the best bridge ID:

* Bridge ID = Priority + MAC address.

Lower is better, both switches have the same priority, but the MAC address of SW2 is lower, which means that SW2 will “win this battle”. SW3 is our loser here which means it will have to block its port, effectively breaking our loop! Take a look at my example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/01/stp-alternate-port-1.png" alt=""><figcaption></figcaption></figure>

If you look at the link between SW2 and SW3, you can see that the Fa1/0 interface of SW3 says **“A”** which stands for **alternate**. An alternate port is blocked! Sometimes the alternate port is called the **ND (Non Designated) port**. By shutting down this interface, we have solved our loop problem.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/01/old-dirty-switch.png" alt=""><figcaption></figcaption></figure>

&#x20;

Because the default priority is 32768, the MAC address is the tie-breaker for selecting the root bridge. What switch do you think will be elected as the root bridge in a production network?

Your brand spanking new switch or that dirty old switch that has been used as a dust collector for the last 8 years?

The old switch probably has a lower MAC address and thus will be elected as the root bridge. Doesn’t sound like a good idea, right? That’s why we can **change the priority** to determine what switch will become the root bridge.

Are you following me so far? Good! You just learned the basics of spanning-tree. Let’s add some more detail to this story…

Let’s continue our spanning-tree story and further enhance your knowledge. If you have played with some Cisco switches before, you might have noticed that every time you plugged in a cable, the led above the interface was orange and after a while became green. What is happening at this moment is that spanning-tree is determining the state of the interface; this is what happens as soon as you plug in a cable:

* The port is in **listening** mode for 15 seconds. In this phase, it will receive and send BPDUs, still, neither learning MAC addresses nor data transmission.
* The port is in **learning** mode for 15 seconds.  We are still sending and receiving BPDUs, but now the switch will also learn MAC addresses. Still no data transmission, though.
* Now we go in **forwarding** mode, and finally, we can start transmitting data!

Here’s a picture to visualize it:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/01/stp-port-states.png" alt=""><figcaption></figcaption></figure>

### Spanning-tree configuration on Cisco switches

Now you have an idea what spanning-tree is about. Let’s take a look at some Cisco switches to see how we can configure them. I will use the same topology that I showed you earlier, but we have different interfaces.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/01/switches-in-triangle2-1.png" alt=""><figcaption></figcaption></figure>

This is the topology we will use. Spanning-tree is enabled by default; let’s start by checking some show commands.

<pre><code><strong>SW1#show spanning-tree
</strong>
VLAN0001
Spanning tree enabled protocol ieee
Root ID    Priority    32769
Address     000f.34ca.1000
Cost        19
Port        19 (FastEthernet0/17)
Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
Address     0011.bb0b.3600
Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/14              Desg FWD 19        128.16   P2p
Fa0/17              Root FWD 19        128.19   P2p
</code></pre>

The show spanning-tree command is the most important show command to remember. There’s quite some stuff here so I’m going to break it down for you!

<pre><code><strong>VLAN0001
</strong><strong>  Spanning tree enabled protocol ieee
</strong></code></pre>

We are looking at the spanning-tree information for VLAN 1. Spanning-tree has multiple versions and the default version on Cisco switches is PVST (Per VLAN spanning-tree). This is the spanning-tree for VLAN 1

<pre><code><strong>Root ID      Priority    32769
</strong><strong>             Address     000f.34ca.1000
</strong><strong>             Cost        19
</strong><strong>             Port        19 (FastEthernet0/17)
</strong></code></pre>

Here you see the information of the root bridge. You can see that it has a priority of 32769 and its MAC address is 000f.34ca.1000. From the perspective of SW1 it has a cost of 19 to reach the root bridge. The port that leads to the root bridge is called the root port and for SW1 this is fa0/17.

<pre><code><strong>Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
</strong><strong>             Address     0011.bb0b.3600
</strong></code></pre>

This part shows us the information about the local switch, SW1 in our case. There’s something funny about the priority here….you can see it shows two things:

* Priority 32769
* Priority 32768 sys-id-ext 1

The sys-id-ext value that you see is the VLAN number. The priority is 32768, but spanning-tree will add the VLAN number, so we end up with a priority value of 32769. Last but not least, we can see the MAC address of SW1, which is 0011.bb0b.3600.

```
Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
```

Here’s some information on the different times that spanning-tree uses:

* Hello time: every 2 seconds, a BPDU is sent.
* Max Age: If we don’t receive BPDUs for 20 seconds, we know something has changed in the network, and we need to re-check the topology.
* Forward Delay: This timer is used for the listening and learning states. We remain in each state for the duration of the forward delay, which is 15 seconds by default.

<pre><code>Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
<strong>Fa0/14              Desg FWD 19        128.16   P2p 
</strong><strong>Fa0/17              Root FWD 19        128.19   P2p
</strong></code></pre>

The last part of the show spanning-tree commands show us the interfaces and their status. SW1 has two interfaces:

• Fa0/14 is a designated port and in (FWD) forwarding mode.\
• Fa0/17 is a root port and in (FWD) forwarding mode.

The `prio.nbr` you see here is the port priority that I explained earlier. We’ll play with this in a bit.

Because only non-root switches have a root port I can conclude that SW1 is a non-root switch. I know that fa0/17 on SW1 leads to the root bridge.

Let’s take a look at SW2 to see what we find:

<pre><code><strong>SW2#show spanning-tree 
</strong>
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     000f.34ca.1000
             Cost        19
             Port        18 (FastEthernet0/16)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

 Bridge ID  Priority    32769 (priority 32768 sys-id-ext 1)
             Address     0019.569d.5700
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/14              Altn BLK 19      128.16   P2p 
Fa0/16              Root FWD 19      128.18   P2p
</code></pre>

What do we see here?

<pre><code><strong>Root ID    Priority    32769
</strong><strong>             Address     000f.34ca.1000
</strong>             Cost        19
<strong>             Port        18 (FastEthernet0/16)
</strong></code></pre>

Here we see information about the root bridge. This information is similar to what we saw on SW1. The root port for SW2 seems to be fa0/16.

<pre><code><strong>Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
</strong><strong>             Address     0019.569d.5700
</strong></code></pre>

This is the information about SW2. The priority is the same as on SW1. Only the MAC address (0019.569d.5700) is different.

<pre><code>Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
<strong>Fa0/14              Altn BLK 19        128.16   P2p 
</strong><strong>Fa0/16              Root FWD 19        128.18   P2p
</strong></code></pre>

This part looks interesting; there are two things we see here:

• Interface fa0/14 is an alternate port and in (BLK) blocking mode.\
• Interface fa0/16 is a root port and in (FWD) forwarding mode.

Last but not least, let’s check SW3:

<pre><code><strong>SW3#show spanning-tree 
</strong>
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     000f.34ca.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000f.34ca.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/14           Desg FWD 19        128.14   P2p 
Fa0/16           Desg FWD 19        128.16   P2p
</code></pre>

Let’s break down what we have here:

<pre><code><strong>Root ID    Priority    32769
</strong><strong>             Address     000f.34ca.1000
</strong><strong>             This bridge is the root
</strong></code></pre>

Bingo… SW3 is the root bridge in this network. We already knew that because SW1 and SW2 are both non-root but this is how we verify it by looking at SW3.

<pre><code><strong>Bridge ID    Priority    32769  (priority 32768 sys-id-ext 1)
</strong><strong>             Address     000f.34ca.1000
</strong></code></pre>

We can also see the MAC address of SW3.

<pre><code>Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
<strong>Fa0/14           Desg FWD 19        128.14   P2p 
</strong><strong>Fa0/16           Desg FWD 19        128.16   P2p
</strong></code></pre>

Both interfaces on SW3 are designated ports and in (FWD) forwarding mode.

You have now seen what the spanning-tree topology looks like. Why was SW3 chosen as the root bridge? We’ll have to verify the bridge ID for the answer:

<pre><code><strong>SW1#show spanning-tree | begin Bridge ID
</strong>  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0011.bb0b.3600
</code></pre>

<pre><code><strong>SW2#show spanning-tree | begin Bridge ID
</strong>  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0019.569d.5700
</code></pre>

<pre><code><strong>SW3#show spanning-tree | begin Bridge ID
</strong>  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000f.34ca.1000
</code></pre>

The priority is the same on all switches (32768), so we have to look at the MAC addresses:

* SW1: 0011.bb0b.3600
* SW2: 0019.569d.5700
* SW3: 000f.34ca.1000

SW3 has the lowest MAC address, so it became root bridge. Why was the fa0/14 interface on SW2 blocked and not the fa0/14 interface on SW1? Once again, we have to look at the bridge identifier. The priority is 32768 on both switches, so we have to compare the MAC address:

* SW1: 0011.bb0b.3600
* SW2: 0019.569d.5700

SW1 has a lower MAC address and thus a better bridge identifier. That’s why SW2 lost this battle and has to shut down its fa0/14 interface.

That’s it! You have now learned how spanning-tree works and how you can check the spanning-tree topology on your Cisco switches. If you enjoyed this lesson, please leave a comment or share it on Facebook or Twitter.
