# Spanning-Tree RootGuard

RootGuard will make sure you don’t accept a certain switch as a root bridge. BPDUs are sent and processed normally but if a switch suddenly sends a BPDU with a superior bridge ID you won’t accept it as the root bridge. Normally SW2 would become the root bridge because it has the best bridge ID, fortunately we have RootGuard on SW3 so it’s not going to happen!

Let me demonstrate this with the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/spanning-tree-bpduguard-topology.png" alt=""><figcaption></figcaption></figure>

Let me show you the configuration by using SW2 and SW3, first I will make sure that SW3 is NOT the root bridge:

<pre><code><strong>SW2(config)#spanning-tree vlan 1 priority 4096
</strong></code></pre>

Now we’ll enable `rootguard` on SW2:

<pre><code><strong>SW2(config)#interface fa0/16
</strong><strong>SW2(config-if)#spanning-tree guard root 
</strong>%SPANTREE-2-ROOTGUARD_CONFIG_CHANGE: Root guard enabled on port FastEthernet0/16.
</code></pre>

We get a nice notification message that it has been enabled. Let’s enable a debug so we can see what is going on:

<pre><code><strong>SW2#debug spanning-tree events 
</strong>Spanning Tree event debugging is on
</code></pre>

Now we’ll upset SW2 by changing the priority to the lowest value possible (0) on SW3. Normally it should now become the root bridge:

<pre><code><strong>SW3(config)#spanning-tree vlan 1 priority 0
</strong></code></pre>

Let’s see what SW2 thinks about this:

```
SW2#
STP: VLAN0001 heard root     1-000f.34ca.1000 on Fa0/16
supersedes  4097-0019.569d.5700
%SPANTREE-2-ROOTGUARD_BLOCK: Root guard blocking port FastEthernet0/16 on VLAN0001.
```

Here goes…SW2 will not accept SW3 as a root bridge. It will block the interface for this VLAN. Here’s another useful command to verify this:

<pre><code><strong>SW2#show spanning-tree inconsistentports 
</strong>
Name                 Interface                Inconsistency
-------------------- ------------------------ ------------------
VLAN0001             FastEthernet0/16         Root Inconsistent

Number of inconsistent ports (segments) in the system : 1
</code></pre>

It’s telling us that `Fastethernet0/16` is inconsistent. `Rootguard` is a useful command to enable on your Core or Distribution layer switches so that the underlying switches will never be elected as a root bridge.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="SW2" %}
```
hostname SW2
!
spanning-tree vlan 1 priority 4096
!
interface FastEthernet0/16
 spanning-tree guard root
!
end
```
{% endtab %}

{% tab title="SW3" %}
```
hostname SW3
!
spanning-tree vlan 1 priority 0
!
end
```
{% endtab %}
{% endtabs %}
