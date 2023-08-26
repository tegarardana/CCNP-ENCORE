# OSPF Path Selection

As you might have learned in CCNA or CCNP, OSPF will use cost as the metric to choose the **shortest path** for each destination, this is true but it’s not entirely correct. OSPF will first look at the “type of path” to make a decision and, secondly look at the metric. This is the preferred path list that OSPF uses:

* Intra-Area (O)
* Inter-Area (O IA)
* External Type 1 (E1)
* NSSA Type 1 (N1)
* External Type 2 (E2)
* NSSA Type 2 (N2)

**After the path selection,** it will look at the lowest-cost path. To give a quick example…when Prefix “X” is learned as an intra-area route (O) and as an inter-area route (O IA), then OSPF will **always** select the intra-area route, even if the inter-area route has a lower cost.

{% hint style="info" %}
Since Cisco IOS release 15.1(2)S, Cisco uses the path selection order from [RFC 3101](https://tools.ietf.org/html/rfc3101), which obsoletes [RFC 1587](https://tools.ietf.org/html/rfc1587). This means that it prefers N1 routes before E1 and N2 over E2 routes. In other words, the preferred path list is O > O IA > N1 > E1 > N2 > E2.
{% endhint %}

I will demonstrate this behavior to you using the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/03/ospf-path-selection-topology.png" alt=""><figcaption></figcaption></figure>

We will create a loopback0 interface on R2 – R7 using the same prefix 1.1.1.1/32 and advertise it in OSPF as following:

* R2: Intra-Area (O)
* R3: Inter-Area (O IA)
* R4: External Type 1 (E1)
* R5: External Type 2 (E2)
* R6: NSSA Type 1 (N1)
* R7: NSSA Type 2 (N2)

We will check R1 to see what path it will prefer. Let’s configure OSPF first:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#router-id 11.11.11.11
</strong><strong>R1(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong><strong>R1(config-router)#network 192.168.13.0 0.0.0.255 area 0
</strong><strong>R1(config-router)#network 192.168.14.0 0.0.0.255 area 0
</strong><strong>R1(config-router)#network 192.168.15.0 0.0.0.255 area 0
</strong><strong>R1(config-router)#network 192.168.16.0 0.0.0.255 area 167
</strong><strong>R1(config-router)#network 192.168.17.0 0.0.0.255 area 167
</strong><strong>R1(config-router)#area 167 nssa
</strong></code></pre>

First, we’ll advertise the correct areas on R1. Don’t forget to make area 167 the NSSA area. Let’s continue with the other routers:

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#router-id 22.22.22.22
</strong><strong>R2(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong><strong>R2(config-router)#network 1.1.1.1 0.0.0.0 area 0
</strong></code></pre>

On R2, we will advertise 1.1.1.1/32 as an intra-area route.

<pre><code><strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#router-id 33.33.33.33
</strong><strong>R3(config-router)#network 192.168.13.0 0.0.0.255 area 0
</strong><strong>R3(config-router)#network 1.1.1.1 0.0.0.0 area 3
</strong></code></pre>

R3 will advertise 1.1.1.1/32 in area 3 to make it an inter-area route.

<pre><code><strong>R4(config)#router ospf 1
</strong><strong>R4(config-router)#router-id 44.44.44.44
</strong><strong>R4(config-router)#network 192.168.14.0 0.0.0.255 area 0       
</strong><strong>R4(config-router)#redistribute connected subnets metric-type 1
</strong></code></pre>

R4 will redistribute prefix 1.1.1.1/32 as an external type 1 route.

<pre><code><strong>R5(config-if)#router ospf 1
</strong><strong>R5(config-router)#router-id 55.55.55.55
</strong><strong>R5(config-router)#network 192.168.15.0 0.0.0.255 area 0
</strong><strong>R5(config-router)#redistribute connected subnets metric-type 2
</strong></code></pre>

R5 will redistribute prefix 1.1.1.1/32 as an external type 2 route.

<pre><code><strong>R6(config)#router ospf 1
</strong><strong>R6(config-router)#router-id 66.66.66.66
</strong><strong>R6(config-router)#network 192.168.16.0 0.0.0.255 area 167       
</strong><strong>R6(config-router)#redistribute connected subnets metric-type 1
</strong><strong>R6(config-router)#area 167 nssa 
</strong></code></pre>

R6 is an NSSA ABR and will advertise 1.1.1.1/32 as an N1 route.

<pre><code><strong>R7(config)#router ospf 1
</strong><strong>R7(config-router)#router-id 77.77.77.77
</strong><strong>R7(config-router)#network 192.168.17.0 0.0.0.255 area 167
</strong><strong>R7(config-router)#redistribute connected subnets metric-type 2
</strong><strong>R7(config-router)#area 167 nssa
</strong></code></pre>

Lastly, R7 will redistribute 1.1.1.1/32, so it shows up as an N2 route.

{% hint style="info" %}
Since I’m creating a loopback interface with the same IP address on router R2-R7, we will have duplicate OSPF router ids. Make sure you make them unique on each router with the **`router-id`** command.
{% endhint %}

Let’s verify our configuration:

<pre><code><strong>R1#show ip ospf neighbor
</strong>
Neighbor ID     Pri   State           Dead Time   Address         Interface
55.55.55.55       1   FULL/BDR        00:00:38    192.168.15.5    FastEthernet0/3
44.44.44.44       1   FULL/BDR        00:00:38    192.168.14.4    FastEthernet0/2
33.33.33.33       1   FULL/BDR        00:00:38    192.168.13.3    FastEthernet0/1
22.22.22.22       1   FULL/BDR        00:00:37    192.168.12.2    FastEthernet0/0
77.77.77.77       1   FULL/BDR        00:00:30    192.168.17.7    FastEthernet0/5
66.66.66.66       1   FULL/BDR        00:00:39    192.168.16.6    FastEthernet0/4
</code></pre>

All OSPF neighbor adjacencies are working. Let’s take a look at the routing table to see what path OSPF has decided to use:

<pre><code><strong>R1#show ip route ospf 
</strong>     1.0.0.0/32 is subnetted, 1 subnets
O       1.1.1.1 [110/2] via 192.168.12.2, 00:07:55, FastEthernet0/0
</code></pre>

Above, you see that R1 has decided to use the path to R2 to reach 1.1.1.1/32. The loopback and FastEthernet interface each have a cost of 1, so the total cost is 2. This path has been selected because it’s an **intra-area route**. What do you think will happen if we increase the cost? Let’s find out:

<pre><code><strong>R2(config)#interface loopback 0
</strong><strong>R2(config-if)#ip ospf cost 1000
</strong></code></pre>

Let’s change the cost to 1000, so this route isn’t very interesting anymore, and check the routing table:

<pre><code><strong>R1#show ip route ospf 
</strong>     1.0.0.0/32 is subnetted, 1 subnets
O       1.1.1.1 [110/1001] via 192.168.12.2, 00:02:02, FastEthernet0/0
</code></pre>

The cost has been increased to 1001, but R1 **still prefers** the path to R2. This is because intra-area paths are preferred over anything else…even if the cost is higher! Let’s shut the interface to R2 to see what path we will take next:

<pre><code><strong>R2(config)#interface loopback0
</strong><strong>R2(config-if)#shutdown
</strong></code></pre>

<pre><code><strong>R1#show ip route ospf 
</strong>     1.0.0.0/32 is subnetted, 1 subnets
O IA    1.1.1.1 [110/2] via 192.168.13.3, 00:00:16, FastEthernet0/1
</code></pre>

With the intra-area route out of the way, R1 prefers the inter-area route to R3…even if we increase the cost, we will stick to this path. Let me show you:

<pre><code><strong>R3(config)#interface loopback 0
</strong><strong>R3(config-if)#ip ospf cost 900
</strong></code></pre>

<pre><code><strong>R1#show ip route ospf 
</strong>     1.0.0.0/32 is subnetted, 1 subnets
O IA    1.1.1.1 [110/901] via 192.168.13.3, 00:00:15, FastEthernet0/1
</code></pre>

Even if I increase the cost to 901, OSPF will stick to the inter-area route over all the other paths. OSPF puts a lot of faith in its path selection. Now you know that path selection is done before the lowest cost selection. Let’s see what OSPF prefers when this inter-area route is no longer reachable:

<pre><code><strong>R3(config)#interface loopback 0
</strong><strong>R3(config-if)#shutdown
</strong></code></pre>

<pre><code><strong>R1#show ip route ospf 
</strong>     1.0.0.0/32 is subnetted, 1 subnets
O E1    1.1.1.1 [110/21] via 192.168.14.4, 00:00:12, FastEthernet0/2
</code></pre>

With the intra-area and inter-area paths out of the way, OSPF will prefer the external type 1 route. Let’s shut it to see what’s next:

<pre><code><strong>R4(config)#interface loopback 0
</strong><strong>R4(config-if)#shutdown
</strong></code></pre>

<pre><code><strong>R1#show ip route ospf 
</strong>     1.0.0.0/32 is subnetted, 1 subnets
O N1    1.1.1.1 [110/21] via 192.168.16.6, 00:00:13, FastEthernet0/4
</code></pre>

The next one is the external type 1 from the NSSA area. Let’s continue:

<pre><code><strong>R6(config)#interface loopback 0
</strong><strong>R6(config-if)#shutdown
</strong></code></pre>

<pre><code><strong>R1#show ip route ospf 
</strong>     1.0.0.0/32 is subnetted, 1 subnets
O E2    1.1.1.1 [110/20] via 192.168.15.5, 00:00:09, FastEthernet0/3
</code></pre>

Now it prefers the external type 2…one more interface to shut:

<pre><code><strong>R5(config)#interface loopback 0
</strong><strong>R5(config-if)#shutdown
</strong></code></pre>

<pre><code><strong>R1#show ip route ospf 
</strong>     1.0.0.0/32 is subnetted, 1 subnets
O N2    1.1.1.1 [110/20] via 192.168.17.7, 00:00:07, FastEthernet0/5
</code></pre>

Last but not least, we will use the external type 2 from the NSSA area.

Keep this in mind before looking at the lowest-cost path…OSPF will first compare the different paths and select one according to the table at the beginning of this lesson. If you have any questions feel free to leave a comment!
