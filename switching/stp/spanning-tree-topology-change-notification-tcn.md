# Spanning-Tree Topology Change Notification (TCN)

How does spanning-tree deal with topology changes? This is a topic that isn’t (heavily) tested on the CCNP SWITCH exam but it’s very important to understand if you deal with a network with a lot of switches.

Let me show you an example so I can explain a couple of things:

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Let’s take a look at the picture above. We have two computers because I need something to fill the MAC address tables of these switches. All switches have the default configuration.

<pre><code><strong>SW3(config)#spanning-tree vlan 1 priority 4096
</strong></code></pre>

<pre><code><strong>SW2(config)#interface fa0/19
</strong><strong>SW2(config-if)#spanning-tree cost 50
</strong></code></pre>

I want SW3 to be the root bridge and the fa0/19 interface of SW2 should be blocked. I’ll show you why in a minute. Let’s see what the state is of all these interfaces:

<pre><code><strong>SW1#show spanning-tree | begin Interface
</strong>Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/14              Desg FWD 19        128.16   P2p 
Fa0/16              Root FWD 19        128.18   P2p
</code></pre>

<pre><code><strong>SW2#show spanning-tree | begin Interface
</strong>Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/2               Desg FWD 19        128.4    P2p 
Fa0/14              Root FWD 19        128.16   P2p 
Fa0/19              Altn BLK 50        128.21   P2p
</code></pre>

<pre><code><strong>SW3#show spanning-tree | begin Interface
</strong>Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/13           Desg FWD 19        128.13   P2p 
Fa0/19           Desg FWD 19        128.19   P2p
</code></pre>

<pre><code><strong>SW4#show spanning-tree | begin Interface
</strong>Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/4               Desg FWD 19        128.4    P2p 
Fa0/16              Desg FWD 19        128.16   P2p 
Fa0/19              Root FWD 19        128.19   P2p
</code></pre>

So here we have all the different interfaces, time to draw a nice picture!

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

Traffic between H1 and H2 will flow from SW2 to SW1, SW3 and then towards SW4. Interface fa0/19 on SW2 has been blocked.

Let’s generate some traffic so the switches learn the MAC addresses of the computers:

<pre><code><strong>C:Documents and SettingsH1>ping 192.168.1.2
</strong>
Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
</code></pre>

A simple ping will generate some frames and the switches will learn the MAC addresses.

In my case these are the MAC addresses for the computers:

* H1: 000c.2928.5c6c
* H2: 000c.29e2.03ba

<pre><code><strong>SW1#show mac address-table dynamic 
</strong>          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    000c.2928.5c6c    DYNAMIC     Fa0/14
   1    000c.29e2.03ba    DYNAMIC     Fa0/16
</code></pre>

<pre><code><strong>SW2#show mac address-table dynamic 
</strong>          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    000c.2928.5c6c    DYNAMIC     Fa0/2
   1    000c.29e2.03ba    DYNAMIC     Fa0/14
</code></pre>

<pre><code><strong>SW3#show mac address-table dynamic 
</strong>          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    000c.2928.5c6c    DYNAMIC     Fa0/13
   1    000c.29e2.03ba    DYNAMIC     Fa0/19
</code></pre>

<pre><code><strong>SW4#show mac address-table dynamic 
</strong>          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    000c.2928.5c6c    DYNAMIC     Fa0/19
   1    000c.29e2.03ba    DYNAMIC     Fa0/4
</code></pre>

We can confirm the traffic path by looking at the MAC address table. I like to use the **show mac address-table dynamic** command so we don’t have to browse through a list of static MAC addresses.

Do you have any idea how long a switch will store a MAC address?

<pre><code><strong>SW1#show mac address-table aging-time 
</strong>Global Aging Time:  300
</code></pre>

If we look at one of the switches we can check the default aging time of the MAC address table. As you can see this is **300 seconds** (5 minutes by default). If a host has been silent for 5 minutes its MAC address will be removed from the table.

Why do we care about aging time? I’ll show you why!

<pre><code><strong>C:Documents and SettingsH1>ping 192.168.1.2 -t
</strong></code></pre>

First I’m going to get some traffic going from H1 to H2. By using ping –t it will run forever.

The next step will be to unplug one of the cables:

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Assume the link between SW1 and SW3 fails. H1 and H2 will be unable to communicate with each other until the fa0/19 interface of SW2 goes into forwarding.

It will take a maximum of 50 seconds for SW2 to move the fa0/19 interface from blocking to listening, learning and finally the forwarding state.

Meanwhile SW2 still has the MAC address of H2 in its MAC address table and will keep forwarding it to SW1 where it will be dropped. It will be impossible for our computers to communicate with each other for 300 seconds until the MAC address tables age out.

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

“Sending Ethernet frames to a place where no frame has gone before doesn’t sound like a good idea if you want to keep your users happy…”

The idea of MAC address tables that age out after 300 seconds works perfectly fine in a stable network but not when the topology changes. Of course there’s a solution to every problem and that’s why spanning-tree has a topology change mechanism.

When a switch detects a change in the network (interface going down or into forwarding state) it will advertise this event to the whole switched network.

When the switches receive this message they will reduce the aging time of the MAC address table from 300 seconds to 15 seconds (this is the forward delay timer). This message is called the TCN (Topology Change Notification).

To take a closer look at the TCN we’ll have to do some debugging…

<pre><code><strong>SW1#debug spanning-tree events 
</strong>Spanning Tree event debugging is on
</code></pre>

<pre><code><strong>SW2#debug spanning-tree events 
</strong>Spanning Tree event debugging is on
</code></pre>

<pre><code><strong>SW3#debug spanning-tree events 
</strong>Spanning Tree event debugging is on
</code></pre>

<pre><code><strong>SW4#debug spanning-tree events 
</strong>Spanning Tree event debugging is on
</code></pre>

I’m going to enable debug spanning-tree events on all switches so you can see this process in action.

Now we will shut the Interface fa0/16 on SW1 to simulate a link failure:

<pre><code><strong>SW1(config)#interface fa0/16
</strong><strong>SW1(config-if)#shutdown
</strong></code></pre>

Here’s what you’ll see:

```
SW1#STP: VLAN0001 sent Topology Change Notice on Fa0/14
```

You will see quite some debug information but somewhere along the lines you’ll see that SW1 is generating a topology change notification and sends it on its fa0/14 interface to SW2. Here’s what you see on SW2:

```
SW2#STP: VLAN0001 Topology Change rcvd on Fa0/14
```

SW2 will throw quite some debug stuff in your face but this is what I was looking for. You can see that it received the topology change notification from SW1. Upon arrival of this topology change notification SW2 will age out its MAC address table in 15 seconds.

What will SW2 do with this information? Look below:

```
SW2#STP: VLAN0001 new root port Fa0/19, cost 69
SW2#STP: VLAN0001 Fa0/19 -> listening
SW2#STP: VLAN0001 Topology Change rcvd on Fa0/14
SW2#STP: VLAN0001 sent Topology Change Notice on Fa0/19
SW2#STP: VLAN0001 Fa0/19 -> learning
SW2#STP: VLAN0001 sent Topology Change Notice on Fa0/19
SW2#STP: VLAN0001 Fa0/19 -> forwarding
```

SW2 decides that fa0/19 is now the new root port and you can see the transition from listening to learning and forwarding mode. It’s also sending a topology change notification towards SW4.

```
SW3#STP: VLAN0001 Topology Change rcvd on Fa0/19
```

SW3 receives a topology change notification on its fa0/19 interface and will reduce its age out timer of the MAC address table to 15 seconds.

```
SW4#STP: VLAN0001 Topology Change rcvd on Fa0/16
SW4#STP: VLAN0001 sent Topology Change Notice on Fa0/19
```

Here we see that SW4 receives the topology change notification from SW2 and as a result it will reduce its age out timer of the MAC address table to 15 seconds. It’s also sending a topology change notification to SW3.

All switches received the topology change notification and set their age out timer to 15 seconds. SW2 doesn’t receive any Ethernet Frames with the MAC address of H2 as the source on its fa0/14 interface and will remove this entry from the MAC address table.

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

Meanwhile the fa0/19 interface on SW2 changed from blocking to listening, learning and forwarding state (50 seconds total). SW2 will now learn the MAC address of H2 on its fa0/19 interface and life is good!

Of course the same thing will happen for the MAC address of H1 on SW4.

Are you following me so far? To keep a long story short…we need the topology change notification to reduce the MAC address table aging timer from 300 seconds to 15 seconds to prevent blackholing traffic in a situation like I just explained to you.

So which switches will send and forward the topology change notifications? In our previous debug you saw a couple of messages but where do we send them and why? Is it flooded to all switches? Let’s check it out!

<figure><img src="../../.gitbook/assets/image (30).png" alt="" width="360"><figcaption></figcaption></figure>

In a normal situation a non-root switch will receive BPDUs on its root port but will never send any BPDUs to the root bridge. When a non-root switch detects a topology change it will generate a topology change notification and send it on its root port towards the root bridge.

When a switch receives the topology change notification it will send a (TCA) topology change acknowledgement on its designated port towards the downstream switch. It will create a topology change notification itself and send it on its root port as well…we will work our way up the tree until we reach the root bridge.

What kind of message is used for the TCN? Take a look at this BPDU:

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

You can see it has a field called BPDU type. This value will change to indicate it’s a topology change notification.

Once the topology change notification reaches the root bridge it will set the TC (topology change) bit in the BPDUs it will send.

<figure><img src="../../.gitbook/assets/image (33).png" alt="" width="358"><figcaption></figcaption></figure>

These BPDUs will be forwarded to all the other switches in our network so they can reduce their aging time of the MAC address table. Switches will receives these messages on both forwarding and blocked ports.

The root bridge will send BPDUs and it will set the flag field to represent the topology change.

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

That’s all there is to it. This is how spanning-tree deals with topology changes in our network. There is one more thing I want to you show you about this mechanism. Take a look at the picture below:

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

As you can see H1 is connected to SW2 on its fa0/2 interface. Let’s see what happens when this interface goes down.

<pre><code><strong>SW2#show spanning-tree interface fa0/2
</strong>
Vlan                Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
VLAN0001            Desg FWD 19        128.4    P2p
</code></pre>

We can see that the fa0/2 interface on SW2 is designated and forwarding. Let’s enable a debug and shut this interface:

<pre><code><strong>SW2#debug spanning-tree events 
</strong>Spanning Tree event debugging is on
</code></pre>

<pre><code><strong>SW2(config)#interface f0/2
</strong><strong>SW2(config-if)#shutdown
</strong></code></pre>

This is what happens on SW2:

```
SW2 STP: VLAN0001 sent Topology Change Notice on Fa0/14
```

Right after shutting down the fa0/2 SW2 generates a topology change notification and sends it away on its root port.

Let’s bring it up again:

<pre><code><strong>SW2(config)#interface f0/2
</strong><strong>SW2(config-if)#no shutdown
</strong></code></pre>

This is what you’ll see:

```
SW2#  STP: VLAN0001 Fa0/2 -> listening
SW2# %LINK-3-UPDOWN: Interface FastEthernet0/2, changed state to up
SW2# %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, SW2# changed state to up
SW2# STP: VLAN0001 Fa0/2 -> learning
SW2# STP: VLAN0001 sent Topology Change Notice on Fa0/14
SW2# STP: VLAN0001 Fa0/2 -> forwarding
```

Once we bring the interface up you can see it goes through the listening and learning state and ends in the forwarding state. The switch generates another topology change notification and sends it on the root port.

What kind of issues could this cause? Imagine we have a network with a LOT of hosts. Each time an interface goes up or down a topology change notification will be generated and ALL switches will set their aging time to 15 seconds. A host will trigger a topology change and if you have a lot of hosts it’s possible that you end up with a network that is in a constant state of “topology changes”.

Here’s a situation that could occur:

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

In the picture above I have a server sending a backup to a LAN backup device. This means we’ll probably have a lot of unicast traffic from the server to the LAN backup device.

Whenever an interface goes down it will generate a topology change notification and as a result all switches will reduce their aging time of the MAC address table to 15 seconds. All the MAC addresses of the devices will be flushed from the MAC address table.

The switches will quickly re-learn the MAC address of the server since its actively sending traffic to the LAN Backup device. If this LAN Backup device is just silently receiving traffic and not sending any traffic itself then there’s no way for the switches to re-learn its MAC address.

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

What happens to unknown MAC unicast traffic? That’s right…it’s flooded on all interfaces except the one where it originated from. As a result this network will be burdened with traffic until our LAN Backup device sends an Ethernet Frame and the switches re-learn its MAC address.

How can we deal with this drama scenario?

[Portfast ](https://networklessons.com/cisco/ccnp-encor-350-401/cisco-portfast-configuration)to the rescue! [Portfast ](https://networklessons.com/cisco/ccnp-encor-350-401/cisco-portfast-configuration)is a Cisco proprietary solution to deal with topology changes. It does two things for us:

* Interfaces with portfast enabled that come up will go to forwarding mode immediately. It will skip the listening and learning state.
* A switch will never generate a topology change notification for an interface that has portfast enabled.

It’s a good idea to enable portfast on interfaces that are connected to hosts because these interfaces are likely to go up and down all the time. Don’t enable portfast on an interface to another hub or switch.
