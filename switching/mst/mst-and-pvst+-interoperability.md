# MST and PVST+ Interoperability

[MST (Multiple Spanning Tree)](https://networklessons.com/cisco/ccnp-encor-350-401/multiple-spanning-tree-mst) and PVST+ (Per VLAN Spanning Tree) both offer loop-free layer two topologies but they each use a different approach:

* MST maps multiple VLANs to an instance, reducing the number of spanning-tree instances.
* PVST+ calculates an instance for each spanning-tree instance.

The two versions are compatible but how should the two behave on the link that connects the two spanning-tree protocols?

PVST+ sends BPDUs for each instance/VLAN so you could let MST process each BPDU separately with the instance that is configured for the VLAN. MST, however, doesn’t have an instance for each VLAN so what kind of BPDUs should it send?

This is an issue…it is possible that two VLANs share the same MST instance but use different root bridges, root ports, and other attributes in the PVST+ domain. Which information should MST send in its BPDUs? There is no way to create a 1:1 mapping between the MST and PVST+ information.

Instead, when an MST region is connected to a PVST+ topology, MST is going to _simulate_ PVST+ with a feature called the **PVST simulation mechanism**. What it does, is that the MST region will send PVST+ BPDUs (one for each VLAN) on the interfaces that are connected to PVST+ switches. These BPDUs all carry the **same information** and advertise the same **root bridge.** The interfaces that connect to the PVST+ topology are called boundary interfaces/ports.

I’m talking about PVST+, but PVRST+ is also supported. However, MST only uses PVST+ features, so there is no performance difference.

Since PVST+ switches now receive BPDUs for each VLAN from MST carrying the same information, they will all make the same decisions when selecting a root bridge, root port, etc.

Where does the information that MST advertises in the simulated PVST+ BPDUs come from?  All MST switches have to agree on the same information, so we use the **information from the IST** for this. **The root bridge in the MST region that is elected as the root bridge for the IST**, will also be used in the PVST+ BPDUs that PVST simulation uses.

What about the other way around? From PVST+ to MST?

MST decides the port role (root, designated, non-designated) for the boundary interface **for all VLANs** only by looking at the information in the **VLAN 1 BPDU**.

Deciding the port role on the boundary interface for all VLANs is risky…after all, it means that we _assume_ that all VLANs use the same root bridge, root ports, etc. We don’t know if the PVST+ domain agrees on the port role for all VLANs that we select on our MST boundary interface. There are, however, some simple checks that MST can use to figure out if PVST+ agrees on the port roles. Let’s take a look at each possible port role.

Designated Port

If MST wants to use the boundary as a designated port. It uses the following check:

```
MST BPDU from IST must be superior compared to each PVST+ BPDU from every VLAN that you receive on the boundary interface.
```

So, MST compares its own IST BPDU to all PVST+ BPDUs (one for each VLAN). If its own BPDU is superior, it knows that PVST+ agrees on the port role and we can safely turn the boundary interface into a designated port. If not, we know that PVST+ doesn’t agree on our port role and we block our boundary interface. The status of the interface shows up as **broken**.

Root Port

The boundary interface can become a root port if the VLAN 1 BPDU that it receives is the best BPDU that it receives on any of its boundary interfaces. This means, that the root bridge for VLAN 1 is located in the PVST+ domain. When this happens, the interface will go into the forwarding state for all VLANs. This is a tricky situation. MST acts as if all VLANs have their root bridge in the PVST+ domain and turns its boundary interface as the root port for all VLANs, but that might not be the case. To counter this, MST uses the following check:

```
PVST+ BPDUs for all VLANs except VLAN 1 must be identical or superior compared to the PVST+ VLAN 1 BPDU.
```

What this means is that MST listens to all PVST+ BPDUs that it receives on its boundary interface and checks if those BPDUs are superior compared to the VLAN 1 BPDU from PVST+. If so, MST can safely turn its boundary interface into a root port. If not, the switch reports an inconsistency error; the interface will be blocked and shows the broken state.

One caveat here is that PVST+ uses the system ID extension, which uses the VLAN number as part of the bridge ID. This makes it impossible to have PVST+ BPDUs from different VLANs with an identical value. Because of this, PVST+ BPDUs require a priority that is **at least 4096 lower** than the PVST+ VLAN 1 BPDU.

Non-designated Port

Last but not least, the boundary interface could be a non-designated port. It becomes a non-designated port if the boundary interface receives a VLAN 1 PVST+ BPDU that is superior to its own MST IST BPDU but not good enough to become a root port. In this case, the boundary interface becomes a non-designated port for all VLANs. MST has to check if the PVST+ domain also thinks this should be a non-designated port and it can quickly check this by listening to all PVST+ BPDUs. When the PVST+ BPDUs are superior to its own BPDU, it can become a non-designated port. Cisco switches don’t do any additional checks. Even if a superior BPDU was received, it would report a PVST inconsistency error and the port would go into broken mode, but since it’s blocked anyway, it doesn’t matter.

It is easiest to configure your network so that the MST region is the root bridge in your network. If your PVST+ domain has the root bridge, then MST will use the same root port for all VLANs. If the root bridge is in your MST region, then you change the cost per VLAN on your PVST+ switches to use different root ports and use a bit of load balancing.

### Configuration

Let’s take a look at all of this in action. There are three switches I use:

<figure><img src="../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

SW1 will use rapid spanning-tree, SW2 and SW3 are going to use MST. We will use three MST instances and six VLANs. Let’s start by configuring the interfaces as trunks and add some VLANs:

<pre><code>SW1, SW2, &#x26; SW3
<strong>(config)#interface range GigabitEthernet 0/1 - 2
</strong><strong>(config-if-range)#switchport trunk encapsulation dot1q 
</strong><strong>(config-if-range)#switchport mode trunk 
</strong></code></pre>

<pre><code>SW1, SW2 &#x26; SW3
<strong>(config)#vlan 10
</strong><strong>(config-vlan)#vlan 20
</strong><strong>(config-vlan)#vlan 30
</strong><strong>(config-vlan)#vlan 40
</strong><strong>(config-vlan)#vlan 50
</strong><strong>(config-vlan)#vlan 60
</strong><strong>(config-vlan)#exit
</strong></code></pre>

We now have a basic configuration that we can use.

#### MST Root Bridge

In the first scenario, I’m going to use the MST region as the root bridge. VLAN 10, 20, and 30 will be mapped to instance 1, VLAN 40, 50, and 60 to instance 2. SW2 is going to be the root bridge for instance 0 and 1:

<pre><code><strong>SW2(config)#spanning-tree mode mst
</strong>
<strong>SW2(config)#spanning-tree mst configuration
</strong><strong>SW2(config-mst)#name REGION1
</strong><strong>SW2(config-mst)#instance 1 vlan 10, 20, 30
</strong><strong>SW2(config-mst)#instance 2 vlan 40, 50, 60
</strong></code></pre>

<pre><code><strong>SW2(config)#spanning-tree mst 0-1 priority 8192
</strong></code></pre>

SW3 will become the root bridge for instance 2:

<pre><code><strong>SW3(config)#spanning-tree mode mst
</strong>
<strong>SW3(config)#spanning-tree mst configuration
</strong><strong>SW3(config-mst)#name REGION1
</strong><strong>SW3(config-mst)#instance 1 vlan 10, 20, 30
</strong><strong>SW3(config-mst)#instance 2 vlan 40, 50, 60
</strong></code></pre>

<pre><code><strong>SW3(config-mst)#spanning-tree mst 2 priority 8192
</strong></code></pre>

SW1 is running rapid spanning-tree with its default priority. Let’s see what happens now when I try to make SW1 the new root bridge for one VLAN:

<pre><code><strong>SW1(config)#spanning-tree vlan 60 priority 4096
</strong></code></pre>

As soon as you do this, this message will show up on SW2 and SW3:

```
SW2#
%SPANTREE-2-PVSTSIM_FAIL: Blocking designated port Gi0/1: Inconsistent inferior PVST BPDU received on VLAN 60, claiming root 4106:fa16.3ec0.f43a
```

```
SW3#
%SPANTREE-2-PVSTSIM_FAIL: Blocking designated port Gi0/1: Inconsistent inferior PVST BPDU received on VLAN 60, claiming root 4106:fa16.3ec0.f43a
```

In order for the boundary interface to be a designated port, the MST IST BPDU must be superior compared to all PVST+ BPDUs which is not the case now.  As a result, the boundary interface is now in the broken state:

<pre><code><strong>SW2#show spanning-tree mst 0
</strong>
##### MST0    vlans mapped:   1-9,11-19,21-29,31-39,41-49,51-59,61-4094
Bridge        address fa16.3e9f.1dac  priority      8192  (8192 sysid 0)
Root          this switch for the CIST
Operational   hello time 2 , forward delay 15, max age 20, txholdcount 6 
Configured    hello time 2 , forward delay 15, max age 20, max hops    20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Desg BKN*20000     128.2    Shr Bound(PVST) *PVST_Inc 
Gi0/2            Desg FWD 20000     128.3    Shr
</code></pre>

<pre><code>
<strong>SW2#show spanning-tree mst 1
</strong>
##### MST1    vlans mapped:   10,20,30
Bridge        address fa16.3e9f.1dac  priority      8193  (8192 sysid 1)
Root          this switch for MST1

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Desg BKN*20000     128.2    Shr Bound(PVST) *PVST_Inc 
Gi0/2            Desg FWD 20000     128.3    Shr
</code></pre>

<pre><code><strong>SW2#show spanning-tree mst 2
</strong>
##### MST2    vlans mapped:   40,50,60
Bridge        address fa16.3e9f.1dac  priority      32770 (32768 sysid 2)
Root          address fa16.3eb2.1388  priority      8194  (8192 sysid 2)
              port    Gi0/2           cost          20000     rem hops 19

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Desg BKN*20000     128.2    Shr Bound(PVST) *PVST_Inc 
Gi0/2            Root FWD 20000     128.3    Shr
</code></pre>

The GigabitEthernet 0/1 interface goes into the BKN (broken) status for all instances/VLANs. In the last output, you can see that even though GigabitEthernet 0/1 is in the broken state, SW2 does recognize SW1 as the root bridge. Let’s increase the priority of SW1 so that SW2 and SW3 do become the root bridges in this topology.

<pre><code><strong>SW1(config)#spanning-tree vlan 60 priority 32768
</strong></code></pre>

This solves our issues with the broken interfaces:

```
SW2#
%SPANTREE-2-PVSTSIM_OK: PVST Simulation inconsistency cleared on port GigabitEthernet0/1.
```

```
SW3#
%SPANTREE-2-PVSTSIM_OK: PVST Simulation inconsistency cleared on port GigabitEthernet0/1.
```

SW1 now sees the MST region as the root bridge. Since all PVST+ BPDUs that SW2 and SW3 receive are inferior compared to the MST IST BPDU, the inconsistency is cleared.

Everything is working correctly now. Let’s take a look at some of the spanning-tree outputs. Here’s VLAN 1 that is mapped to instance 0 where SW2 is the root bridge:

<pre><code><strong>SW1#show spanning-tree vlan 1 
</strong>
VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    8192
             Address     fa16.3e9f.1dac
             Cost        4
             Port        2 (GigabitEthernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     fa16.3ec0.f43a
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Root FWD 4         128.2    Shr Peer(STP) 
Gi0/2               Altn BLK 4         128.3    Shr Peer(STP)
</code></pre>

Above we see that GigabitEthernet 0/1 is the root port. Here’s the output of VLAN 10 which is mapped to instance 1 with SW2 as the root bridge:

<pre><code><strong>SW1#show spanning-tree vlan 10
</strong>
VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    8192
             Address     fa16.3e9f.1dac
             Cost        4
             Port        2 (GigabitEthernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     fa16.3ec0.f43a
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Root FWD 4         128.2    Shr Peer(STP) 
Gi0/2               Altn BLK 4         128.3    Shr Peer(STP)
</code></pre>

The output is the same as VLAN 1. Let’s also take a look at VLAN 60 which is mapped to instance 2 where SW3 is the root bridge:

<pre><code><strong>SW1#show spanning-tree vlan 60
</strong>
VLAN0040
  Spanning tree enabled protocol rstp
  Root ID    Priority    8192
             Address     fa16.3e9f.1dac
             Cost        4
             Port        2 (GigabitEthernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32828  (priority 32768 sys-id-ext 60)
             Address     fa16.3ec0.f43a
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Root FWD 4         128.2    Shr Peer(STP) 
Gi0/2               Altn BLK 4         128.3    Shr Peer(STP)
</code></pre>

The output above is interesting to see. As explained earlier, MST maps all VLANs to the IST, so even VLAN 60 (and 40+50) share the same information, even though SW3 is the root bridge for these VLANs. SW1 sees SW2 as the root bridge for these VLANs.

Having the MST region as the root bridge is a best practice. It’s simple to maintain, and it allows load balancing. For example, let’s say we want the PVST+ domain to use another root port for VLAN 60. We can achieve this by playing with the cost:

<pre><code><strong>SW1(config)#interface GigabitEthernet 0/2
</strong><strong>SW1(config-if)#spanning-tree vlan 60 cost 1
</strong></code></pre>

SW1 now uses GigabitEthernet 0/2 as its root port to reach VLAN 60:

<pre><code><strong>SW1#show spanning-tree vlan 60
</strong>
VLAN0040
  Spanning tree enabled protocol rstp
  Root ID    Priority    8192
             Address     fa16.3e9f.1dac
             Cost        1
             Port        3 (GigabitEthernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32828  (priority 32768 sys-id-ext 60)
             Address     fa16.3ec0.f43a
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Altn BLK 4         128.2    Shr Peer(STP) 
Gi0/2               Root FWD 1         128.3    Shr Peer(STP)
</code></pre>

GigabitEthernet 0/2 is now the root port for VLAN 60. All other VLANs still use GigabitEthernet 0/1 as their root port.

#### PVST+ Root Bridge

It is possible to have the root bridge in the PVST+ domain. Let’s give this a try:

```
SW1(config)#spanning-tree vlan 1 priority 4096
```

Once I change the priority for VLAN 1 on SW1, SW2 starts complaining:

`SW2#`\
`%SPANTREE-2-PVSTSIM_FAIL: Blocking root port Gi0/1: Inconsistent inferior PVST BPDU received on VLAN 10, claiming root 32778:fa16.3ec0.f43a`

This is because of the root port check that MST does. To become a root port, all PVST+ BPDUs except for VLAN 1 need to be identical or superior compared to the PVST+ VLAN 1 BPDU. That’s not the case here, since VLAN 10-60 still have a default priority of 32768. Right now, SW2 puts its boundary interface in the broken state:

<pre><code><strong>SW2#show spanning-tree mst 0
</strong>
##### MST0    vlans mapped:   1-9,11-19,21-29,31-39,41-49,51-59,61-4094
Bridge        address fa16.3e9f.1dac  priority      8192  (8192 sysid 0)
Root          address fa16.3ec0.f43a  priority      4097  (4096 sysid 1)
              port    Gi0/1           path cost     20000    
Regional Root this switch
Operational   hello time 2 , forward delay 15, max age 20, txholdcount 6 
Configured    hello time 2 , forward delay 15, max age 20, max hops    20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Root BKN*20000     128.2    Shr Bound(PVST) *PVST_Inc 
Gi0/2            Desg FWD 20000     128.3    Shr
</code></pre>

<pre><code><strong>SW2#show spanning-tree mst 1
</strong>
##### MST1    vlans mapped:   10,20,30
Bridge        address fa16.3e9f.1dac  priority      8193  (8192 sysid 1)
Root          this switch for MST1

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Mstr BKN*20000     128.2    Shr Bound(PVST) *PVST_Inc 
Gi0/2            Desg FWD 20000     128.3    Shr
</code></pre>

<pre><code><strong>SW2#show spanning-tree mst 2
</strong>
##### MST2    vlans mapped:   40,50,60
Bridge        address fa16.3e9f.1dac  priority      32770 (32768 sysid 2)
Root          address fa16.3eb2.1388  priority      8194  (8192 sysid 2)
              port    Gi0/2           cost          20000     rem hops 19

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Mstr BKN*20000     128.2    Shr Bound(PVST) *PVST_Inc 
Gi0/2            Root FWD 20000     128.3    Shr
</code></pre>

To fix this, we have to lower the priority on all VLANs of SW1, except for VLAN 1:

<pre><code><strong>SW1(config)#spanning-tree vlan 10,20,30,40,50,60 priority 0
</strong></code></pre>

This satisfies the root port check on SW2:

```
SW2#
%SPANTREE-2-PVSTSIM_OK: PVST Simulation inconsistency cleared on port GigabitEthernet0/1
```

Things are looking good now:

<pre><code><strong>SW2#show spanning-tree mst 0
</strong>
##### MST0    vlans mapped:   1-9,11-19,21-29,31-39,41-49,51-59,61-4094
Bridge        address fa16.3e9f.1dac  priority      8192  (8192 sysid 0)
Root          address fa16.3ec0.f43a  priority      4097  (4096 sysid 1)
              port    Gi0/1           path cost     20000    
Regional Root this switch
Operational   hello time 2 , forward delay 15, max age 20, txholdcount 6 
Configured    hello time 2 , forward delay 15, max age 20, max hops    20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Root FWD 20000     128.2    Shr Bound(PVST) 
Gi0/2            Desg FWD 20000     128.3    Shr 
</code></pre>

<pre><code><strong>SW2#show spanning-tree mst 1
</strong>
##### MST1    vlans mapped:   10,20,30
Bridge        address fa16.3e9f.1dac  priority      8193  (8192 sysid 1)
Root          this switch for MST1

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Mstr FWD 20000     128.2    Shr Bound(PVST) 
Gi0/2            Desg FWD 20000     128.3    Shr 
</code></pre>

<pre><code><strong>SW2#show spanning-tree mst 2
</strong>
##### MST2    vlans mapped:   40,50,60
Bridge        address fa16.3e9f.1dac  priority      32770 (32768 sysid 2)
Root          address fa16.3eb2.1388  priority      8194  (8192 sysid 2)
              port    Gi0/2           cost          20000     rem hops 19

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Mstr FWD 20000     128.2    Shr Bound(PVST) 
Gi0/2            Root FWD 20000     128.3    Shr
</code></pre>

Above you can see that the boundary interface is now in the forwarding state for all VLANs.

### Conclusion

In this lesson, you have learned how MST and PVST+ (or PVRST+) are compatible.

MST and PV(R)ST+ are compatible, but it’s not very straightforward since you can’t convert their information 1:1. To work around this, MST uses PVST+ compatibility mode and sends a BPDU for each VLAN on its link (boundary interface) to the PVST+ domain. The information in each BPDU is based on the IST.

MST uses the information from the PVST+ VLAN 1 BPDU to decide the role of the boundary interface (root, designated or non-designated) for all VLANs. This introduces a problem since the PVST+ domain might not agree on the port role. To work around this, MST has a couple of checks:

* Designated port:
  * _MST BPDU from IST must be superior compared to each PVST+ BPDU from every VLAN that you receive on the boundary interface._
* Root port:
  * _PVST+ BPDUs for all VLANs except VLAN 1 must be identical or superior compared to the PVST+ VLAN 1 BPDU._
  * Because of the extended system ID, it’s impossible to have a BPDU with the same value. This means the priority of all non-VLAN 1 BPDUs has to be at least 4096 lower than the PVST+ VLAN 1 BPDU.

I hope you enjoyed this lesson, if you have any questions feel free to leave a reply.
