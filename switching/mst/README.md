# MST

By default, Cisco Catalyst Switches run PVST+ or Rapid PVST+ (Per VLAN Spanning Tree). This means that each VLAN is mapped to a single spanning tree instance. When you have 20 VLANs, it means there are 20 instances of spanning-tree.

Is this a problem? Like always…it depends.

Let’s take a look at an example:

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

Take a look at the topology above. We have three switches and a lot of VLANs. There are 201 VLANs in total. If we are running PVST or Rapid PVST, we have 201 different calculations for each VLAN. This requires a lot of CPU power and memory.

When SW2 is the root bridge for VLAN 100 – 200 and SW3 for VLAN 201 – 300, our spanning-tree topologies will look like this:

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

SW2 is the root bridge for VLAN 100 up to VLAN 200. This means that the GigabitEthernet0/1 interface on SW1 or the GigabitEthernet 0/0 interface of SW3 will be blocked. I’ll have 100 spanning-tree calculations, but they all look the same for these VLANs…

The same thing applies to VLAN 201 – 300. SW3 is the root bridge for VLAN 201 up to 300. The GigabitEthernet 0/0 interface on SW1 or SW2 will be blocked for all these VLANs.

Two different outcomes, but I still have 200 different instances of spanning tree running. That’s a waste of CPU cycles and memory.

**MST (Multiple Spanning Tree)** will solve this issue. Instead of calculating a spanning tree for each VLAN, we can use **instances** and map VLANs to each instance. For the network above, I could do something like this:

* Instance 1: VLAN 100 – 200
* Instance 2: VLAN 201 – 300

Sounds logical, right? Only two spanning tree calculations (instances) are required for all these VLANs.

MST works with the concept of **regions.** Switches that are configured to use MST need to find out if their neighbors are running MST.

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

When switches have the **same attributes,** they will be in the same region. It’s possible to have one or more regions. Here are the attributes that need to match:

* MST configuration name.
* MST configuration revision number.
* MST instance to VLAN mapping table.

When switches have the **same attributes** configured, they will be in the same region. If the attributes are not the same, the switch is seen as being at the boundary of the region. It can be connected to another MST region but also talk to a switch running another version of spanning-tree.

The **MST configuration name** is just something you can make up, it’s used to identify the MST region. The **MST configuration revision number** is also something you can make up and the idea behind this number is that you can change the number whenever you change your configuration. It doesn’t matter what you pick as long as it’s the same on all switches within the MST region. VLANs will be mapped to an instance by using the **MST instance to VLAN mapping table**. This is something we have to do ourselves.

Within the MST region, we will have one instance of spanning tree that will create a loop-free topology **within the region.** When you configure MST, there is always one default instance used to calculate the topology within the region. We call this the **IST (Internal Spanning Tree).** By default, Cisco will use **instance 0** to run the IST. The IST runs **rapid spanning-tree**.

<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

I could create instances 1 for VLAN 100 – 200 and 2 for VLAN 201 – 300. Depending on which switch will become the root bridge for each instance, a different port will be blocked. It could look like this:

<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

The switch outside the MST region doesn’t see what the MST region looks like. For this switch, it’s like it’s talking to one big switch or a ‘black box’:

<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

If you want to know the details of how MST and PVST+ work together, check out our [MST and PVST+ interoperability lesson](https://networklessons.com/cisco/ccnp-encor-350-401/mst-pvst-interoperability).  Let’s have some fun with the configuration.

### MST Configuration

I will use the following topology:

<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

We’ll start with a single MST region with the following attributes:

* MST configuration name: “NETWORKLESSONS”
* MST configuration revision number: 1 (this is just a number that I made up)
* MST instance to VLAN mapping table:
  * Instance 1: VLAN 10, 20, and 30.
  * Instance 2: VLAN 40, 50, and 60.

This is what we will do:

<pre><code><strong>SW1(config)#spanning-tree mode mst
</strong></code></pre>

<pre><code><strong>SW2(config)#spanning-tree mode mst
</strong></code></pre>

<pre><code><strong>SW3(config)#spanning-tree mode mst
</strong></code></pre>

This is how we enable MST on our switches. Let’s look at the default MST instance:

<pre><code><strong>SW1#show spanning-tree mst configuration
</strong>Name      []
Revision  0     Instances configured 1

Instance  Vlans mapped
--------  ---------------------------------------------------------------------
0         1-4094
-------------------------------------------------------------------------------
</code></pre>

<pre><code><strong>SW2#show spanning-tree mst configuration 
</strong>Name      []
Revision  0     Instances configured 1

Instance  Vlans mapped
--------  ---------------------------------------------------------------------
0         1-4094
-------------------------------------------------------------------------------
</code></pre>

<pre><code><strong>SW3#show spanning-tree mst configuration 
</strong>Name      []
Revision  0     Instances configured 1

Instance  Vlans mapped
--------  ---------------------------------------------------------------------
0         1-4094
-------------------------------------------------------------------------------
</code></pre>

We can use the **show spanning-tree mst configuration** command to see the MST instances. I haven’t created any additional instances so only instance 0 is available. You can see that all VLANs are currently mapped to instance 0. Let’s see what else we can find:hiero

<pre><code><strong>SW1#show spanning-tree mst
</strong>
##### MST0    vlans mapped:   1-4094
Bridge        address 5254.0010.370d  priority      32768 (32768 sysid 0)
Root          address 5254.0001.50c4  priority      32768 (32768 sysid 0)
              port    Gi0/0           path cost     0        
Regional Root address 5254.0001.50c4  priority      32768 (32768 sysid 0)
                                      internal cost 20000     rem hops 19
Operational   hello time 2 , forward delay 15, max age 20, txholdcount 6 
Configured    hello time 2 , forward delay 15, max age 20, max hops    20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Root FWD 20000     128.1    P2p 
Gi0/1            Desg FWD 20000     128.2    P2p
</code></pre>

You can also use the `show spanning-tree mst` command. We can see the VLAN mapping but also information about the root bridge. Before we can add more instances we have to do our chores…time to add some VLANs and configure the links between the switches as trunks:

<pre><code><strong>SW1(config)#interface GigabitEthernet 0/0
</strong><strong>SW1(config-if)#switchport trunk encapsulation dot1q
</strong><strong>SW1(config-if)#switchport mode trunk
</strong><strong>SW1(config)#interface GigabitEthernet 0/1
</strong><strong>SW1(config-if)#switchport trunk encapsulation dot1q
</strong><strong>SW1(config-if)#switchport mode trunk
</strong></code></pre>

<pre><code><strong>SW2(config)#interface GigabitEthernet 0/0
</strong><strong>SW2(config-if)#switchport trunk encapsulation dot1q
</strong><strong>SW2(config-if)#switchport mode trunk
</strong><strong>SW2(config)#interface GigabitEthernet 0/1
</strong><strong>SW2(config-if)#switchport trunk encapsulation dot1q
</strong><strong>SW2(config-if)#switchport mode trunk
</strong></code></pre>

<pre><code><strong>SW3(config)#interface GigabitEthernet 0/0
</strong><strong>SW3(config-if)#switchport trunk encapsulation dot1q
</strong><strong>SW3(config-if)#switchport mode trunk
</strong><strong>SW3(config)#interface GigabitEthernet 0/1
</strong><strong>SW3(config-if)#switchport trunk encapsulation dot1q
</strong><strong>SW3(config-if)#switchport mode trunk
</strong></code></pre>

That takes care of the trunks. Here are the VLANs:

<pre><code>SW1, SW2 &#x26; SW3:
<strong>(config)#vlan 10
</strong><strong>(config-vlan)#vlan 20
</strong><strong>(config-vlan)#vlan 30
</strong><strong>(config-vlan)#vlan 40
</strong><strong>(config-vlan)#vlan 50
</strong><strong>(config-vlan)#vlan 60
</strong><strong>(config-vlan)#exit
</strong></code></pre>

Now we can configure MST and the instances:

<pre><code><strong>SW1(config)#spanning-tree mst configuration 
</strong><strong>SW1(config-mst)#name NETWORKLESSONS
</strong><strong>SW1(config-mst)#revision 1
</strong><strong>SW1(config-mst)#instance 1 vlan 10,20,30
</strong><strong>SW1(config-mst)#instance 2 vlan 40,50,60
</strong><strong>SW1(config-mst)#exit
</strong></code></pre>

<pre><code><strong>SW2(config)#spanning-tree mst configuration 
</strong><strong>SW2(config-mst)#name NETWORKLESSONS
</strong><strong>SW2(config-mst)#revision 1
</strong><strong>SW2(config-mst)#instance 1 vlan 10,20,30
</strong><strong>SW2(config-mst)#instance 2 vlan 40,50,60
</strong><strong>SW2(config-mst)#exit
</strong></code></pre>

<pre><code><strong>SW3(config)#spanning-tree mst configuration 
</strong><strong>SW3(config-mst)#name NETWORKLESSONS
</strong><strong>SW3(config-mst)#revision 1
</strong><strong>SW3(config-mst)#instance 1 vlan 10,20,30
</strong><strong>SW3(config-mst)#instance 2 vlan 40,50,60
</strong><strong>SW3(config-mst)#exit
</strong></code></pre>

This is how we configure MST. First, you need the **spanning-tree mst configuration** command to enter the configuration of MST. We set the name by using the **name** command. Don’t forget to set a **revision number** and map the instances with the **instance** command. Let’s verify our work:

<pre><code><strong>SW1#show spanning-tree mst configuration
</strong>Name      [NETWORKLESSONS]
Revision  1     Instances configured 3

Instance  Vlans mapped
--------  ---------------------------------------------------------------------
0         1-9,11-19,21-29,31-39,41-49,51-59,61-4094
1         10,20,30
2         40,50,60
-------------------------------------------------------------------------------
</code></pre>

We can use the show spanning-tree mst configuration command to verify our configuration. You can see that we now have two instances. The VLANs are mapped to instances 1 and 2. All the other VLANs are still mapped to instance 0.

So far, so good. Let’s play some more with MST and change the root bridge:

<figure><img src="../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

I want to ensure that SW1 is the root bridge within our region. We’ll have to change the priority for the IST (Internal Spanning Tree):

<pre><code><strong>SW1(config)#spanning-tree mst 0 priority 4096
</strong></code></pre>

This is how I change the priority for MST instance 0. Let’s verify this:

<pre><code><strong>SW1#show spanning-tree mst
</strong>
##### MST0    vlans mapped:   1-9,11-19,21-29,31-39,41-49,51-59,61-4094
Bridge        address 5254.0010.370d  priority      4096  (4096 sysid 0)
Root          this switch for the CIST
Operational   hello time 2 , forward delay 15, max age 20, txholdcount 6 
Configured    hello time 2 , forward delay 15, max age 20, max hops    20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Desg FWD 20000     128.1    P2p 
Gi0/1            Desg FWD 20000     128.2    P2p
</code></pre>

Here you can see that SW1 is the root bridge for the IST. It says **CIST (Common and Internal Spanning Tree)**.

Let’s take a look at the interfaces:

<pre><code><strong>SW1#show spanning-tree mst 0 | begin Interface
</strong>Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Desg FWD 20000     128.1    P2p 
Gi0/1            Desg FWD 20000     128.2    P2p 
</code></pre>

<pre><code><strong>SW2#show spanning-tree mst 0 | begin Interface
</strong>Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Root FWD 20000     128.1    P2p 
Gi0/1            Desg FWD 20000     128.2    P2p 
</code></pre>

<pre><code><strong>SW3#show spanning-tree mst 0 | begin Interface
</strong>Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Root FWD 20000     128.1    P2p 
Gi0/1            Altn BLK 20000     128.2    P2p 
</code></pre>

Now we know the state of all interfaces. Let’s draw a picture, so we know what the IST looks like:

<figure><img src="../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

Now I want to make some changes to instance 1 so SW2 will be the root bridge:

<pre><code><strong>SW2(config)#spanning-tree mst 1 priority 4096
</strong></code></pre>

We’ll change the priority on SW2 for instance 2.

<pre><code><strong>SW2#show spanning-tree mst 1
</strong>
##### MST1    vlans mapped:   10,20,30
Bridge        address 5254.0001.50c4  priority      4097 (4096 sysid 1)
Root          this switch for MST1

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Desg FWD 20000     128.2    P2p 
</code></pre>

This command proves that SW2 is the root bridge for instance 1. Let’s check the interfaces:

<pre><code><strong>SW1#show spanning-tree mst 1 | begin Interface
</strong>
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Root FWD 20000     128.1    P2p 
Gi0/1            Desg FWD 20000     128.2    P2p
</code></pre>

```
SW2#show spanning-tree mst 1 | begin Interface
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Desg FWD 20000     128.1    P2p 
Gi0/1            Desg FWD 20000     128.2    P2p
```

```
SW3#show spanning-tree mst 1 | begin Interface
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Altn BLK 20000     128.1    P2p 
Gi0/1            Root FWD 20000     128.2    P2p
```

This is what instance 1 looks like. Let’s turn that into a nice picture:

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

Here’s a picture of instance 1 to show you the port roles. Note that this topology looks different than the one for instance 0.

Last but not least I’m now going to make some changes for instance 2:

<pre><code><strong>SW3(config)#spanning-tree mst 2 priority 4096
</strong></code></pre>

SW3 will become the root bridge for instance 2.

<pre><code><strong>SW3#show spanning-tree mst 2
</strong>
##### MST2    vlans mapped:   40,50,60
Bridge        address 5254.0016.36f5  priority      4098  (4096 sysid 2)
Root          this switch for MST2

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Desg FWD 20000     128.1    P2p 
Gi0/1            Desg FWD 20000     128.2    P2p
</code></pre>

SW3 is now the root bridge for instance 2. Let’s look at the interfaces:

```
SW1#show spanning-tree mst 2 | begin Interface
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Altn BLK 20000     128.1    P2p 
Gi0/1            Root FWD 20000     128.2    P2p 
```

```
SW2#show spanning-tree mst 2 | begin Interface
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Desg FWD 20000     128.1    P2p 
Gi0/1            Root FWD 20000     128.2    P2p 
```

```
SW3#show spanning-tree mst 2 | begin Interface
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Desg FWD 20000     128.1    P2p 
Gi0/1            Desg FWD 20000     128.2    P2p 
```

And we can draw another topology picture:

<figure><img src="../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

Let’s compare instances 1 and 2 next to each other:

<figure><img src="../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

On the left side, you see instance 1 and on the right side is instance 2.

By changing the root bridge per instance, we end up with different topologies:

* Instance 1: GigabitEthernet0/0 on SW3 is blocked for VLAN 10, 20, and 30.
* Instance 2: GigabitEthernet0/0 on SW1 is blocked for VLAN 40, 50, and 60.

What happens when I add another switch that is running PVST to our topology? Let’s find out!

<figure><img src="../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

I’ll configure the GigabitEthernet0/2 interfaces on SW2 and SW3 as trunks:

<pre><code><strong>SW2(config)#interface GigabitEthernet0/2
</strong><strong>SW2(config-if)#switchport trunk encapsulation dot1q
</strong><strong>SW2(config-if)#switchport mode trunk
</strong></code></pre>

<pre><code><strong>SW3(config)#interface GigabitEthernet0/2
</strong><strong>SW3(config-if)#switchport trunk encapsulation dot1q
</strong><strong>SW3(config-if)#switchport mode trunk
</strong></code></pre>

And we’ll configure the GigabitEthernet0/0 and 0/1 interfaces of SW4 as trunks:

<pre><code><strong>SW3(config)#interface GigabitEthernet0/0
</strong><strong>SW3(config-if)#switchport trunk encapsulation dot1q 
</strong><strong>SW3(config-if)#switchport mode trunk
</strong></code></pre>

<pre><code><strong>SW3(config)#interface GigabitEthernet0/1
</strong><strong>SW3(config-if)#switchport trunk encapsulation dot1q
</strong><strong>SW3(config-if)#switchport mode trunk
</strong></code></pre>

PVST is the default spanning-tree mode on Cisco switches so we don’t have to change this:

<pre><code><strong>SW4#show running-config all | include spanning-tree mode
</strong>spanning-tree mode pvst
</code></pre>

Let’s add VLAN 10, 20, 30, 40, 50, and 60 on SW4:

<pre><code><strong>SW4(config)#vlan 10
</strong><strong>SW4(config-vlan)#vlan 20
</strong><strong>SW4(config-vlan)#vlan 30
</strong><strong>SW4(config-vlan)#vlan 40
</strong><strong>SW4(config-vlan)#vlan 50
</strong><strong>SW4(config-vlan)#vlan 60
</strong><strong>SW4(config-vlan)#exit
</strong></code></pre>

I want to ensure that we have a trunk to SW2 and SW3 and that SW4 knows about all the VLANs. Let’s see what SW4 thinks of all this:

<pre><code><strong>SW4#show spanning-tree vlan 1
</strong>
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    4096
             Address     5254.0010.370d
             Cost        4
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5254.0017.2f95
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p 
Gi0/1               Altn BLK 4         128.2    P2p 
</code></pre>

This is what SW4 sees about VLAN 1. Keep in mind this VLAN was mapped to instance 0. It sees SW1 as the root bridge, and you can see which port is in forwarding and blocking mode.

<pre><code><strong>SW4#show spanning-tree vlan 10
</strong>
VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    4096
             Address     5254.0010.370d
             Cost        4
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     5254.0017.2f95
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p 
Gi0/1               Altn BLK 4         128.2    P2p 
</code></pre>

Here’s VLAN 10, which is mapped to instance 1. SW4 sees SW2 as the root bridge for this VLAN even though we configured SW3 as the root bridge for instance 2. This is perfectly normal because **MST will only advertise BPDUs from the IST to the outside world**. We won’t see any information from instance 1 or instance 2 on SW4.

Let’s check VLAN 40:

<pre><code><strong>SW4#show spanning-tree vlan 40
</strong>
VLAN0040
  Spanning tree enabled protocol ieee
  Root ID    Priority    4096
             Address     5254.0010.370d
             Cost        4
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32808  (priority 32768 sys-id-ext 40)
             Address     5254.0017.2f95
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p 
Gi0/1               Altn BLK 4         128.2    P2p
</code></pre>

VLAN 40 is mapped to instance 2, but you can see that SW4 sees SW1 as the root bridge. SW4 receives the same BPDU for all VLANs.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="SW1" %}
```
hostname SW1
!
spanning-tree mode mst
!
spanning-tree mst configuration
name NETWORKLESSONS
revision 1
instance 1 vlan 10, 20, 30
instance 2 vlan 40, 50, 60
!
spanning-tree mst 0 priority 4096
!
interface GigabitEthernet0/0
switchport trunk encapsulation dot1q
switchport mode trunk
!
interface GigabitEthernet0/1
switchport trunk encapsulation dot1q
switchport mode trunk
negotiation auto
!
end
```
{% endtab %}

{% tab title="SW2" %}
```
hostname SW2
!
spanning-tree mode mst
!
spanning-tree mst configuration
name NETWORKLESSONS
revision 1
instance 1 vlan 10, 20, 30
instance 2 vlan 40, 50, 60
!
spanning-tree mst 1 priority 4096
!
interface GigabitEthernet0/0
switchport trunk encapsulation dot1q
switchport mode trunk
!
interface GigabitEthernet0/1
switchport trunk encapsulation dot1q
switchport mode trunk
!
interface GigabitEthernet0/2
switchport trunk encapsulation dot1q
switchport mode trunk
!
end
```
{% endtab %}

{% tab title="SW3" %}
```
hostname SW3
!
spanning-tree mode mst
!
spanning-tree mst configuration
name NETWORKLESSONS
revision 1
instance 1 vlan 10, 20, 30
instance 2 vlan 40, 50, 60
!
spanning-tree mst 2 priority 4096
!
interface GigabitEthernet0/0
switchport trunk encapsulation dot1q
switchport mode trunk
!
interface GigabitEthernet0/1
switchport trunk encapsulation dot1q
switchport mode trunk
!
interface GigabitEthernet0/2
switchport trunk encapsulation dot1q
switchport mode trunk
!
end
```
{% endtab %}

{% tab title="SW4" %}
```
hostname SW4
!
spanning-tree mode pvst
!
interface GigabitEthernet0/0
switchport trunk encapsulation dot1q
switchport mode trunk
!
interface GigabitEthernet0/1
switchport trunk encapsulation dot1q
switchport mode trunk
!
end
```
{% endtab %}
{% endtabs %}
