# Rapid Spanning-Tree Configuration

In a previous lesson I explained the differences between classic and rapid spanning-tree and how rapid spanning-tree works. If you haven’t seen it before, I would recommend to [look at it first](https://networklessons.com/cisco/ccnp-encor-350-401/rapid-spanning-tree-rstp) before diving in the configuration.

Having said that, let’s look at the configuration. This is the topology that I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-example.png" alt=""><figcaption></figcaption></figure>

This is the topology I’m going to use. SW1 will be the root bridge in my example. First we have to enable rapid spanning-tree:

<pre><code><strong>SW1(config)#spanning-tree mode rapid-pvst
</strong></code></pre>

<pre><code><strong>SW2(config)#spanning-tree mode rapid-pvst
</strong></code></pre>

<pre><code><strong>SW3(config)#spanning-tree mode rapid-pvst
</strong></code></pre>

That’s it…just one command will enable rapid spanning tree on our switches. The implementation of rapid spanning tree is **rapid-pvst**. We are calculating a rapid spanning tree for each VLAN.

First I want to show you the sync mechanism:

<pre><code><strong>SW1(config)#interface fa0/14
</strong><strong>SW1(config-if)#shutdown
</strong><strong>SW1(config)#interface f0/17
</strong><strong>SW1(config-if)#shutdown
</strong></code></pre>

I’m going to shut both interfaces on SW1 to start with.

<pre><code><strong>SW1#debug spanning-tree events 
</strong>Spanning Tree event debugging is on
</code></pre>

<pre><code><strong>SW2#debug spanning-tree events 
</strong>Spanning Tree event debugging is on
</code></pre>

<pre><code><strong>SW3#debug spanning-tree events 
</strong>Spanning Tree event debugging is on
</code></pre>

Second step is to enable debug on all the switches.

<pre><code><strong>SW1(config)#interface fa0/14
</strong><strong>SW1(config-if)#no shutdown
</strong></code></pre>

I’m going to bring the fa0/14 interface back to the land of the living on SW1. Here’s what we see:

```
SW1#
setting bridge id (which=3) prio 4097 prio cfg 4096 sysid 1 (on) id 1001.0011.bb0b.3600
RSTP(1): initializing port Fa0/14
RSTP(1): Fa0/14 is now designated
RSTP(1): transmitting a proposal on Fa0/14
```

The fa0/14 interface on SW1 will be blocked and it’ll send a proposal to SW2.

```
SW2#
RSTP(1): initializing port Fa0/14
RSTP(1): Fa0/14 is now designated
RSTP(1): transmitting a proposal on Fa0/14
RSTP(1): updt roles, received superior bpdu on Fa0/14 
RSTP(1): Fa0/14 is now root port
```

Apparently SW2 thought it was the root bridge because it says it received a superior BPDU on its fa0/14 interface. It changes its fa0/14 interface to root port.

```
SW2# RSTP(1): syncing port Fa0/16
```

The fa0/16 interface on SW2 will go into sync mode. This is the interface that connects to SW3.

```
SW2#  RSTP(1): synced Fa0/14
RSTP(1): transmitting an agreement on Fa0/14 as a response to a proposal
```

SW2 will respond to SW1 its proposal by sending an agreement.

```
SW1# RSTP(1): received an agreement on Fa0/14
%LINK-3-UPDOWN: Interface FastEthernet0/14, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/14, changed state to up
```

SW1 receives the agreement from SW2 and interface fa0/14 will go into forwarding.

```
SW2# RSTP(1): transmitting a proposal on Fa0/16
```

SW2 will send a proposal to SW3.

```
SW3# RSTP(1): transmitting an agreement on Fa0/16 as a response to a proposal
```

SW3 will respond to the proposal of SW2 and send an agreement.

```
SW2# RSTP(1): received an agreement on Fa0/16
%LINK-3-UPDOWN: Interface FastEthernet0/14, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/14, changed state to up
```

SW2 receives the agreement from SW3 and the interface will go into forwarding. That’s all there to is it…a quick number of handshakes and the interfaces will move to forwarding without the use of any timers. Let’s continue!

<pre><code><strong>SW1(config)#interface fa0/17
</strong><strong>SW1(config-if)#no shutdown
</strong></code></pre>

I’m going to enable this interface so that connectivity is fully restored. Let’s look at an overview:

<pre><code><strong>SW1#show spanning-tree 
</strong>
VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    4097
             Address     0011.bb0b.3600
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4097   (priority 4096 sys-id-ext 1)
             Address     0011.bb0b.3600
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/14              Desg FWD 19        128.16   P2p 
Fa0/17              Desg FWD 19        128.19   P2p
</code></pre>

We can verify that SW1 is the root bridge. This show command also reveals that we are running rapid spanning tree. Note that the link type is **p2p**. This is because my FastEthernet interfaces are in full duplex by default. Let’s run the same command on the other two switches:

<pre><code><strong>SW2#show spanning-tree 
</strong>
VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    4097
             Address     0011.bb0b.3600
             Cost        19
             Port        16 (FastEthernet0/14)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    8193   (priority 8192 sys-id-ext 1)
             Address     0019.569d.5700
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/14              Root FWD 19        128.16   P2p 
Fa0/16              Desg FWD 19        128.18   P2p
</code></pre>

<pre><code><strong>SW3#show spanning-tree 
</strong>
VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    4097
             Address     0011.bb0b.3600
             Cost        19
             Port        14 (FastEthernet0/14)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000f.34ca.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/14           Root FWD 19        128.14   P2p 
Fa0/16           Altn BLK 19        128.16   P2p
</code></pre>

Here are SW2 and SW3. Nothing new here, it’s the same information as classic spanning tree. Here’s what the topology looks like now:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-example-with-ports.png" alt=""><figcaption></figcaption></figure>

Let’s add another link between SW2 and SW3 to see if this influences our topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-example-second-link.png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>SW2#show spanning-tree | begin Interface
</strong>Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/14              Root FWD 19        128.16   P2p 
Fa0/16              Desg FWD 19        128.18   P2p 
Fa0/17              Desg FWD 19        128.19   P2p
</code></pre>

<pre><code><strong>SW3#show spanning-tree | begin Interface
</strong>Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/14           Root FWD 19        128.14   P2p 
Fa0/16           Altn BLK 19        128.16   P2p 
Fa0/17           Altn BLK 19        128.17   P2p
</code></pre>

Nothing spectacular, we just have another designated port on SW2 and another alternate port on SW3. Let me add that alternate port to the topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-two-alternate-ports.png" alt=""><figcaption></figcaption></figure>

So far the topology with rapid spanning-tree looks the same as with classic spanning-tree. Now let me show you something you haven’t seen before. I will add a hub between SW2 and SW3:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-hub.png" alt=""><figcaption></figcaption></figure>

Now take a look again at the interfaces:

<pre><code><strong>SW2#show spanning-tree | begin Interface
</strong>
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/14               Root FWD 19        128.5    P2p
Fa0/16               Desg FWD 100       128.3    Shr 
Fa0/17               Back BLK 100       128.4    Shr 
</code></pre>

<pre><code><strong>SW3#show spanning-tree | begin Interface 
</strong>

Interface           Role Sts Cost      Prio.Nbr Type 
--------- -------- --------------------------------
Fa0/14               Root FWD 19        128.5    P2p
Fa0/16               Altn BLK 100       128.3    Shr 
Fa0/17               Altn BLK 100       128.4    Shr
</code></pre>

Here’s something new. SW2 has a backup port. Because of the hub in the middle SW2 and SW3 will hear their own BPDUs.

You can also see that the link type is **shr (shared)**. That’s because the hub causes these switches to switch their interfaces to half duplex. Here’s the topology picture again:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-backup-port-hub.png" alt=""><figcaption></figcaption></figure>

You probably won’t ever see the backup port on a production network since hubs are scarce now but if you see it, you’ll know why…

BPDUs are sent every two seconds (hello time) and if you want to prove this you can take a look at a debug:

<pre><code><strong>SW2#debug spanning-tree bpdu
</strong></code></pre>

You can use the debug spanning-tree bpdu command to view BPDUs are are sent or received.

```
SW2#
STP: VLAN0001 rx BPDU: config protocol = rstp, packet from FastEthernet0/14  , linktype IEEE_SPANNING , enctype 2, encsize 17 
STP: enc 01 80 C2 00 00 00 00 11 BB 0B 36 10 00 27 42 42 03 
STP: Data     000002023C10010011BB0B36000000000010010011BB0B360080100000140002000F00
STP: VLAN0001 Fa0/14:0000 02 02 3C 10010011BB0B3600 00000000 10010011BB0B3600 8010 0000 1400 0200 0F00
RSTP(1): Fa0/14 repeated msg
RSTP(1): Fa0/14 rcvd info remaining 6
RSTP(1): sending BPDU out Fa0/16
RSTP(1): sending BPDU out Fa0/17
STP: VLAN0001 rx BPDU: config protocol = rstp, packet f
```

You will see the contents of the BPDU like above. It’s not very useful if you want to see the content of the BPDU but it does show us that SW2 is receiving BPDUs and sending them on its interfaces.

If you do want to look at the contents of a BPDU I recommend you to use wireshark. It shows everything in a nice structured way.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/wireshark-rapid-spanning-tree-bpdu-300x199.png" alt=""><figcaption></figcaption></figure>

You don’t have to capture a BPDU yourself if you don’t feel like. The wireshark website has many pre-recorded packet captures.

&#x20;

&#x20;

Let’s get rid of the hub and do something else…I’m going to simulate a link failure between SW1 and SW3 to see how rapid spanning tree deals with this.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-link-broken.png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>SW1(config)#interface fa0/17 
</strong><strong>SW1(config-if)#shutdown
</strong></code></pre>

First I’m going to shut the fa0/17 interface on SW1.

```
SW3#
RSTP(1): updt rolesroot port Fa0/14 is going down
RSTP(1): Fa0/16 is now root port
```

SW3 realized something is wrong with the root port almost immediately and will change the fa0/16 interface from alternate port to root port. This is the equivalent of UplinkFast for classic spanning tree but it’s enabled by default for rapid spanning tree.

<pre><code><strong>SW1(config)#interface fa0/17
</strong><strong>SW1(config-if)#no shutdown
</strong></code></pre>

Let’s restore connectivity before we continue.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-link-broken-2.png" alt=""><figcaption></figcaption></figure>

Let’s simulate an indirect link failure. The classic spanning-tree has backbone fast and a similar mechanism is enabled by default for rapid spanning tree.

<pre><code><strong>SW1(config)#interface fa0/14
</strong><strong>SW1(config-if)#shutdown
</strong></code></pre>

Shutting down this interface will simulate an indirect link failure for SW3.

```
SW2#
RSTP(1): updt roles, root port Fa0/14 going down
RSTP(1): we become the root bridge
RSTP(1): updt roles, received superior bpdu on Fa0/16 
RSTP(1): Fa0/16 is now root port
```

```
SW3#
03:41:29: RSTP(1): updt rolessuperior bpdu on Fa0/16 (synced=0)
03:41:29: RSTP(1): Fa0/16 is now designated
```

SW2 believes it’s the root bridge until it receives a superior BPDU from SW3. This happens within the blink of an eye.

<pre><code><strong>SW1(config)#interface fa0/14
</strong><strong>SW1(config-if)#no shutdown
</strong></code></pre>

Let’s get rid of the shutdown command and continue…let’s look at the edge ports:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-computer-connected.png" alt=""><figcaption></figcaption></figure>

I added H1 and it’s connected to the fa0/2 interface of SW2.  Let’s see how rapid spanning tree deals with interfaces connected to other devices:

<pre><code><strong>SW2(config)#interface fa0/2
</strong><strong>SW2(config-if)#no shutdown
</strong>RSTP(1): initializing port Fa0/2
RSTP(1): Fa0/2 is now designated
RSTP(1): transmitting a proposal on Fa0/2
%LINK-3-UPDOWN: Interface FastEthernet0/2, changed state to up
RSTP(1): transmitting a proposal on Fa0/2
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up
RSTP(1): transmitting a proposal on Fa0/2
RSTP(1): transmitting a proposal on Fa0/2
RSTP(1): transmitting a proposal on Fa0/2
RSTP(1): transmitting a proposal on Fa0/2
RSTP(1): transmitting a proposal on Fa0/2
RSTP(1): transmitting a proposal on Fa0/2
RSTP(1): transmitting a proposal on Fa0/2
RSTP(1): Fa0/2 fdwhile Expired
</code></pre>

You see that it sends a bunch of proposals from the sync mechanism towards the computer. After a while they will expire. The port will end up in forwarding state anyway but it takes a while.

<pre><code><strong>SW2(config-if)#spanning-tree portfast 
</strong>%Warning: portfast should only be enabled on ports connected to a single
 host. Connecting hubs, concentrators, switches, bridges, etc... to this
 interface  when portfast is enabled, can cause temporary bridging loops.
 Use with CAUTION

%Portfast has been configured on FastEthernet0/2 but will only
 have effect when the interface is in a non-trunking mode.
</code></pre>

You have to tell rapid spanning tree that the interface connecting the computer is an edge port. The word “edge” makes sense; it’s the border of our spanning tree topology. Enable portfast and you are ready to go.

<pre><code><strong>SW2(config)#interface fa0/2
</strong><strong>SW2(config-if)#shutdown
</strong><strong>SW2(config-if)#no shutdown
</strong></code></pre>

I’ll bring the interface up and down.

```
SW2#
RSTP(1): initializing port Fa0/2
RSTP(1): Fa0/2 is now designated
*Mar  1 04:08:32.931: %LINK-3-UPDOWN: Interface FastEthernet0/2, changed state to up
```

The interface will go to forwarding immediately. Our switch knows that this is the edge of the spanning tree and we don’t have to send proposals to it. The last thing we have to look at is compatibility…

I’m going to change SW2 to PVST mode. SW1 and SW3 will remain at rapid-PVST:

```
SW2(config)#spanning-tree mode pvst
```

Here’s what we see:

```
SW2(config)#
RSTP(1): updt roles, non-tracked event
setting bridge id (which=3) prio 8193 prio cfg 8192 sysid 1 (on) id 2001.0019.569d.5700
set portid: VLAN0001 Fa0/2: new port id 8004
STP: VLAN0001 Fa0/2 ->jump to forwarding from blocking
set portid: VLAN0001 Fa0/14: new port id 8010
STP: VLAN0001 Fa0/14 -> listening
set portid: VLAN0001 Fa0/16: new port id 8012
STP: VLAN0001 Fa0/16 -> listening^Z
STP: VLAN0001 heard root  4097-0011.bb0b.3600 on Fa0/16 supersedes  8193-0019.569d.5700
STP: VLAN0001 new root is 4097, 0011.bb0b.3600 on port Fa0/16, cost 38
STP: VLAN0001 new root port Fa0/14, cost 19
STP: VLAN0001 Fa0/14 -> learning
STP: VLAN0001 Fa0/16 -> learning
STP: VLAN0001 sent Topology Change Notice on Fa0/14
STP: VLAN0001 Fa0/14 -> forwarding
STP: VLAN0001 Fa0/16 -> forwarding
```

SW2 will throw some information at you. You can see that it receives BPDUs from the root bridge and that the interfaces will have to go through the listening and learning state. When the switches that are talking rapid spanning tree receive a BPDU from the classic spanning tree they will generate classic spanning tree BPDUs themselves so everything keeps working.

```
SW1#show spanning-tree | begin Interface
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/14              Desg FWD 19        128.16   P2p Peer(STP) 
Fa0/17              Desg FWD 19        128.19   P2p

SW2#show spanning-tree | begin Interface
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/2               Desg FWD 19        128.4    P2p Edge 
Fa0/14              Root FWD 19        128.16   P2p 
Fa0/16              Desg FWD 19        128.18   P2p
```

```
SW3#show spanning-tree | begin Interface
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/14           Root FWD 19        128.14   P2p 
Fa0/16           Altn BLK 19        128.16   P2p Peer(STP)
```

We can verify this by looking at the interfaces again. All switches still agree on the port states and everything will function as it should be!

That’s all there is about rapid spanning-tree. The configuration is pretty simple but I hope the debugs and show commands helped to understand exactly how everything works.
