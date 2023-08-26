# STP Tuning

## Cisco Portfast Configuration

Portfast is a Cisco proprietary solution to deal with [spanning-tree topology changes](https://networklessons.com/cisco/ccnp-encor-350-401/spanning-tree-topology-change-notification-tcn). If you don’t know how spanning-tree reacts to topology changes then I highly recommend you to [read this lesson](https://networklessons.com/cisco/ccnp-encor-350-401/spanning-tree-topology-change-notification-tcn) before you continue reading. It helps to truly understand why we need portfast.

Portfast does two things for us:

• Interfaces with portfast enabled that come up will go to forwarding mode immediately, the interface will skip the listening and learning state.\
• A switch will never generate a topology change notification for an interface that has portfast enabled.

It’s a good idea to enable portfast on interfaces that are connected to hosts because these interfaces are likely to go up and down all the time. Don’t enable portfast on an interface to another hub or switch.

Let’s take a look at the difference of an interface with and without portfast. I’ll be using the following topology for this:

<figure><img src="../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

I have two switches and one host connected to SW1. The only reason I have two switches is so SW1 has another switch that it can send topology notification changes to. Let’s look at the without portfast scenario first…

### Portfast disabled

To see the interesting stuff I will enable a debug on SW1:

<pre><code><strong>SW1#debug spanning-tree events
</strong>Spanning Tree event debugging is on
</code></pre>

Once I plug in the cable to connect the host to SW1 this is what happens:

```
SW1#
STP: VLAN0001 Fa0/1 -> listening
STP: VLAN0001 Fa0/1 -> learning
STP: VLAN0001 Fa0/1 -> forwarding
```

This is just normal spanning-tree behavior, it walks through the listening and learning states and ends up in forwarding.

Each time I unplug the cable, spanning-tree will generate a topology change notification. There’s a nice command that you can use to check how many have been sent so far:

<pre><code><strong>SW1#show spanning-tree detail
</strong>
 VLAN0001 is executing the ieee compatible Spanning Tree protocol
  Bridge Identifier has priority 32768, sysid 1, address 0019.569d.5700
  Configured hello time 2, max age 20, forward delay 15
  Current root has priority 32769, address 0011.bb0b.3600
  Root port is 26 (FastEthernet0/24), cost of root path is 19
  Topology change flag not set, detected flag not set
  Number of topology changes 5 last change occurred 00:02:09 ago
          from FastEthernet0/1
  Times:  hold 1, topology change 35, notification 2
          hello 2, max age 20, forward delay 15
  Timers: hello 0, topology change 0, notification 0, aging 300
</code></pre>

As you can see there have been 5 topology changes so far on VLAN 1. Let’s unplug the cable to the host to see what happens:

```
SW1#
STP: VLAN0001 sent Topology Change Notice on Fa0/24
```

Spanning-tree will send a topology change notification on the interface towards SW2 and the counter will increase:

<pre><code><strong>SW1#show spanning-tree detail | include changes
</strong>  Number of topology changes 6 last change occurred 00:01:12 ago
</code></pre>

In short, everytime we unplug the cable the switch will generate a TCN. Let’s see the difference when we enable portfast…

### Portfast enabled

All we have to do is enable portfast on the FastEthernet 0/1 interface that connects our host:

<pre><code><strong>SW1(config)#interface FastEthernet 0/1
</strong><strong>SW1(config-if)#spanning-tree portfast
</strong>%Warning: portfast should only be enabled on ports connected to a single
 host. Connecting hubs, concentrators, switches, bridges, etc... to this
 interface  when portfast is enabled, can cause temporary bridging loops.
 Use with CAUTION

%Portfast has been configured on FastEthernet0/1 but will only
 have effect when the interface is in a non-trunking mode.
</code></pre>

We get a big warning that portfast shouldn’t be used on interfaces that connect to other switches etc.

There is also a global command “**spanning-tree portfast default**” that will enable portfast on all interfaces that are in access mode. The result will be the same but it saves you from enabling it on each interface seperately.

Let’s connect our host again:

```
SW1#
STP: VLAN0001 Fa0/1 ->jump to forwarding from blocking
```

Great, the interface skips the listening and learning state and goes to forwarding immediately. Also, the switch will no longer generate topology change notifications when you unplug this cable anymore.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of SW1.
{% endtab %}

{% tab title="SW1" %}
```
hostname SW1
!
interface FastEthernet0/1
 spanning-tree portfast
!
end
```
{% endtab %}
{% endtabs %}

I hope this has been helpful to understand portfast, if you enjoyed this lesson, please share it with your friends or colleagues.
