# OSPF Distribute-List Filtering

OSPF supports a number of methods to filter routes but it is more restrictive compared to distance vector routing protocols like RIP or EIGRP.

As a link-state routing protocol OSPF uses LSAs to build its LSDB (Link State Database). Routers will run the SPF algorithm to find the shortest path to each destination, the topology in the LSDB has to be the same on all routers or SPF will fail.

However OSPF routers only know what the topology looks like _within the area_. They don’t know what the topology looks like for other areas. For inter-area routes OSPF only knows the prefix and the ABR (Area Border Router) to reach it.

You could say that OSPF acts like a distance vector routing protocol for inter-area routes. It only knows the metric (distance) and the ABR to get there (vector).

Unlike RIP or EIGRP, OSPF doesn’t advertise routes but LSAs so if we want to filter something we’ll have to filter the advertisement of LSAs.

Since the LSDB within the area has to be the same **we can’t filter LSAs within the area**, we can however filter routes from entering the routing table. Filtering LSAs between areas on an ABR or ASBR is no problem.

In this lesson I’ll show you how we can filter routes from entering the routing table within the area. In other lessons I will explain how to [filter type 3 LSAs](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-abr-type-3-lsa-filtering-on-cisco-ios) and [type 5 LSAs](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-lsa-type-5-filtering-on-cisco-ios).

Here’s the topology I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/ospf-three-routers-single-area.png" alt=""><figcaption></figcaption></figure>

Nothing fancy, we have three routers running OSPF in the same area. R1 has a loopback interface that is advertised in OSPF, we’ll see if we can filter this network.

## Configuration

Here’s the OSPF configuration:

<pre><code><strong>R1#show running-config | section ospf
</strong>router ospf 1
 network 1.1.1.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
</code></pre>

<pre><code><strong>R2#show running-config | section ospf
</strong>router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
</code></pre>

<pre><code><strong>R3#show running-config | section ospf
</strong>router ospf 1
 network 192.168.23.0 0.0.0.255 area 0
</code></pre>

Let’s verify if R2 and R3 have learned 1.1.1.1 /32:

<pre><code><strong>R2#show ip route ospf
</strong>
      1.0.0.0/32 is subnetted, 1 subnets
O        1.1.1.1 [110/2] via 192.168.12.1, 00:00:27, FastEthernet0/0
</code></pre>

<pre><code><strong>R3#show ip route ospf
</strong>
      1.0.0.0/32 is subnetted, 1 subnets
O        1.1.1.1 [110/3] via 192.168.23.2, 00:00:28, FastEthernet0/0
O     192.168.12.0/24 [110/2] via 192.168.23.2, 00:00:28, FastEthernet0/0
</code></pre>

Let’s see if we can get rid of this network on R3:

<pre><code><strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#distribute-list ?
</strong>  &#x3C;1-199>      IP access list number
  &#x3C;1300-2699>  IP expanded access list number
  WORD         Access-list name
  gateway      Filtering incoming updates based on gateway
  prefix       Filter prefixes in routing updates
  route-map    Filter prefixes based on the route-map
</code></pre>

We can use a distribute-list for this, to keep it simple I’ll combine it with an access-list;

<pre><code><strong>R3(config-router)#distribute-list R1_L0 in
</strong></code></pre>

When we want to remove something from the routing table we have to apply it inbound. The outbound distribute-list is used for LSA type 5 filtering.

Let’s create that access-list:

<pre><code><strong>R3(config)#ip access-list standard R1_L0
</strong><strong>R3(config-std-nacl)#deny host 1.1.1.1    
</strong><strong>R3(config-std-nacl)#permit any
</strong></code></pre>

It will now be gone from the routing table:

<pre><code><strong>R3#show ip route 1.1.1.1
</strong>% Network not in table
</code></pre>

As you can see it’s gone…it’s still in the LSDB though:

<pre><code><strong>R3#show ip ospf database router 192.168.12.1
</strong>
            OSPF Router with ID (192.168.23.3) (Process ID 1)

		Router Link States (Area 0)

  LS age: 664
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 192.168.12.1
  Advertising Router: 192.168.12.1
  LS Seq Number: 80000003
  Checksum: 0xF14F
  Length: 48
  Number of Links: 2

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 1.1.1.1
     (Link Data) Network Mask: 255.255.255.255
      Number of MTID metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 192.168.12.2
     (Link Data) Router Interface address: 192.168.12.1
      Number of MTID metrics: 0
       TOS 0 Metrics: 1
</code></pre>

You have to be very careful if you use this command. If you are not careful you can end up in a scenario where you blackhole some traffic. For example, let’s see what happens when I filter this network on R2 instead of R3. Let’s remove the distribute-list on R3:

<pre><code><strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#no distribute-list R1_L0 in
</strong></code></pre>

Now I will add it to R2:

<pre><code><strong>R2(config)#ip access-list standard R1_L0
</strong><strong>R2(config-std-nacl)#deny host 1.1.1.1
</strong><strong>R2(config-std-nacl)#permit any
</strong>
<strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#distribute-list R1_L0 in
</strong></code></pre>

R2 now no longer has it in its routing table:

<pre><code><strong>R2#show ip route 1.1.1.1
</strong>% Network not in table
</code></pre>

However the LSA is still flooded to R3:

<pre><code><strong>R3#show ip route ospf 
</strong>
      1.0.0.0/32 is subnetted, 1 subnets
O        1.1.1.1 [110/3] via 192.168.23.2, 00:02:45, FastEthernet0/0
O     192.168.12.0/24 [110/2] via 192.168.23.2, 00:02:45, FastEthernet0/0
</code></pre>

Once R3 tries to reach this network we will have a problem:

<pre><code><strong>R3#ping 1.1.1.1
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
U.U.U
Success rate is 0 percent (0/5)
</code></pre>

R3 will forward these packets to R2 which drops it.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip cef
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
router ospf 1
 network 1.1.1.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip cef
!
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
interface FastEthernet1/0
 ip address 192.168.23.2 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
 distribute-list R1_L0 in
!
ip access-list standard R1_L0
 deny   1.1.1.1
 permit any
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ip cef
!
interface FastEthernet0/0
 ip address 192.168.23.3 255.255.255.0
!
router ospf 1
 network 192.168.23.0 0.0.0.255 area 0
!
ip access-list standard R1_L0
 deny   1.1.1.1
 permit any
!
end
```
{% endtab %}
{% endtabs %}

That’s all there is to it, you have now seen how you can filter routes within your OSPF area. Make sure you also check my other two lessons on OSPF filtering:

* [OSPF LSA type 3 filtering](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-abr-type-3-lsa-filtering-on-cisco-ios)
* [OSPF LSA type 5 filtering](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-lsa-type-5-filtering-on-cisco-ios)

If you have any questions, feel free to leave a comment!
