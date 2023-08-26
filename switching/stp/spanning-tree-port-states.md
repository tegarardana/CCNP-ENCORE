# Spanning-Tree Port States

If you have played with some Cisco switches before you might have noticed that every time you plug in a cable the led above the interface was orange and after a while became green. What is happening at this moment is that spanning tree is determining the state of the interface.

This is what happens as soon as you plug in a cable:

* **Listening state**: Only a root or designated port will move to the listening state. The non-designated port will stay in the blocking state.No data transmission occurs at this state for 15 seconds just to make sure the topology doesn’t change in the meantime. After the listening state we move to the learning state.
* **Learning state:** At this moment the interface will process Ethernet frames by looking at the source MAC address to fill the mac-address-table. Ethernet frames however are not forwarded to the destination. It takes 15 seconds to move to the next state called the forwarding state.
* **Forwarding state:** This is the final state of the interface and finally the interface will forward Ethernet frames so that we have data transmission!

When a port is not a designated or root port it will be in **blocking mode**.

This means it takes 30 seconds in total to move from listening to forwarding…that’s not really fast right? This will happen on **all interfaces** on the switch.

When an interface is in blocking mode and the topology changes, it’s possible that an interface that is currently in blocking mode has to move to the forwarding state. When this is the case, the blocking mode will last for 20 seconds before it moves to the listening state. This means that it takes 20 (blocking) + 15 (listening) + 15 (learning) = 50 seconds before the interface is in the forwarding state.

30 seconds is a long time right? Any modern PC with a SSD drive boots faster than that. Here’s an overview of the different port states:

| State      | Forward Frames | Learn MAC Addresses | Duration   |
| ---------- | -------------- | ------------------- | ---------- |
| Blocking   | No             | No                  | 20 seconds |
| Listening  | No             | No                  | 15 seconds |
| Learning   | No             | Yes                 | 15 seconds |
| Forwarding | Yes            | Yes                 | –          |

So what does this look like on an actual Cisco switch? Let me show you an example of an interface that is connected to a router. I just unplugged and plugged the cable (or do a”shut” and “no shut”) and the first time we run the show command it looks like this:

<pre><code><strong>SW1#show spanning-tree vlan
</strong>
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0019.569d.5700
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0019.569d.5700
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/1               Desg LIS 19        128.4    P2p
</code></pre>

You can see that the role of the port is designated and the status is listening. Keep refreshing this show command and after \~ 15 seconds it looks like this:

<pre><code><strong>SW1#show spanning-tree vlan 1
</strong>
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0019.569d.5700
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0019.569d.5700
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/1               Desg LRN 19        128.4    P2p
</code></pre>

It has moved to the learning state and after another \~ 15 seconds it looks like this:

<pre><code><strong>SW1#show spanning-tree vlan 1
</strong>
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0019.569d.5700
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0019.569d.5700
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 15

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/1               Desg FWD 19        128.4    P2p
</code></pre>

Very nice, you just witnessed an interface moving through the different spanning tree port states. A better method to see the changes is by enabling a debug:

<pre><code><strong>SW1#debug spanning-tree events
</strong>Spanning Tree event debugging is on
</code></pre>

When we disable and enable the interface again you can see it moving through the spanning tree port states in realtime:

```
SW1#
00:14:57: STP: VLAN0001 Fa0/1 -> listening
00:15:12: STP: VLAN0001 Fa0/1 -> learning
00:15:27: STP: VLAN0001 Fa0/1 -> forwarding
```

That’s pretty neat right? I hope this lesson has helped you to understand the spanning tree port states! If you have any questions, feel free to leave a comment.
