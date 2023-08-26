# OSPF LSA Type 5 Filtering

In previous lessons I explained how you can filter routes [within the OSPF area](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-distribute-list-filtering) and how you can[ filter type 3 LSA](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-abr-type-3-lsa-filtering-on-cisco-ios)s. This time we’ll take a look how you can filter type 5 LSAs using three different methods.

Here’s the topology we will use for this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/ospf-lsa-type-5-filtering-topology.png" alt=""><figcaption></figcaption></figure>

Above we have three routers in two different areas. R1 has some loopback interfaces that we will redistribute into OSPF. We’ll use these to play with some of the filtering techniques.

## Configuration

Here’s the OSPF configuration of all routers:

<pre><code><strong>R1#show running-config | section ospf
</strong>router ospf 1
 redistribute connected subnets
 network 192.168.12.0 0.0.0.255 area 0
</code></pre>

<pre><code><strong>R2#show running-config | section ospf
</strong>router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 1
</code></pre>

<pre><code><strong>R3#show running-config | section ospf
</strong>router ospf 1
 network 192.168.23.0 0.0.0.255 area 1
</code></pre>

R1 is using the **redistribute connected subnets** command to get the networks on the loopback interfaces in OSPF. Let’s see if R2 and R3 have these networks in their routing table:

<pre><code><strong>R2#show ip route ospf 
</strong>
      172.16.0.0/32 is subnetted, 4 subnets
O E2     172.16.0.1 [110/20] via 192.168.12.1, 00:00:03, FastEthernet0/0
O E2     172.16.1.1 [110/20] via 192.168.12.1, 00:00:03, FastEthernet0/0
O E2     172.16.2.1 [110/20] via 192.168.12.1, 00:00:03, FastEthernet0/0
O E2     172.16.3.1 [110/20] via 192.168.12.1, 00:00:03, FastEthernet0/0
</code></pre>

<pre><code><strong>R3#show ip route ospf 
</strong>
      172.16.0.0/32 is subnetted, 4 subnets
O E2     172.16.0.1 [110/20] via 192.168.23.2, 00:00:07, FastEthernet0/0
O E2     172.16.1.1 [110/20] via 192.168.23.2, 00:00:07, FastEthernet0/0
O E2     172.16.2.1 [110/20] via 192.168.23.2, 00:00:07, FastEthernet0/0
O E2     172.16.3.1 [110/20] via 192.168.23.2, 00:00:07, FastEthernet0/0
O IA  192.168.12.0/24 [110/2] via 192.168.23.2, 00:04:25, FastEthernet0/0
</code></pre>

Everything is there. Now let’s see if we can filter these…

### Distribute-list Filtering

The first method is the distribute-list. We can use this on the ASBR to filter certain networks from entering the area. Let’s configure one to get rid of 172.16.0.1 /32:

<pre><code><strong>R1(config)#ip access-list standard R1_L0
</strong><strong>R1(config-std-nacl)#deny host 172.16.0.1
</strong><strong>R1(config-std-nacl)#permit any
</strong>
<strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#distribute-list R1_L0 out
</strong></code></pre>

We will use an outbound distribute-list with an access-list that matches the network (host route). Let’s see if it works:

<pre><code><strong>R2#show ip route ospf 
</strong>
      172.16.0.0/32 is subnetted, 3 subnets
O E2     172.16.1.1 [110/20] via 192.168.12.1, 00:10:12, FastEthernet0/0
O E2     172.16.2.1 [110/20] via 192.168.12.1, 00:10:12, FastEthernet0/0
O E2     172.16.3.1 [110/20] via 192.168.12.1, 00:10:12, FastEthernet0/0
</code></pre>

<pre><code><strong>R3#show ip route ospf
</strong>
      172.16.0.0/32 is subnetted, 3 subnets
O E2     172.16.1.1 [110/20] via 192.168.23.2, 00:10:12, FastEthernet0/0
O E2     172.16.2.1 [110/20] via 192.168.23.2, 00:10:12, FastEthernet0/0
O E2     172.16.3.1 [110/20] via 192.168.23.2, 00:10:12, FastEthernet0/0
O IA  192.168.12.0/24 [110/2] via 192.168.23.2, 00:14:30, FastEthernet0/0
</code></pre>

The entry has dissapeared from the routing tables of R2 and R3.

### Redistribution with Route-Map

The previous example works but there’s a better solution. Why not prevent certain routes from being redistributed in the first place? Technically this isn’t “filtering” but it works very well.

Let’s see what the current redistribute command looks like now:

<pre><code><strong>R1#show running-config | include redistribute
</strong> redistribute connected subnets
</code></pre>

We’ll create a route-map that denies 172.16.1.1 /32 from being redistributed while we allow everything else. When it’s finished we’ll attach it to the redistribute command above:

<pre><code><strong>R1(config)#ip access-list standard R1_L1
</strong><strong>R1(config-std-nacl)#permit host 172.16.1.1
</strong>
<strong>R1(config)#route-map CONNECTED_TO_OSPF deny 10
</strong><strong>R1(config-route-map)#match ip address R1_L1
</strong>
<strong>R1(config)#route-map CONNECTED_TO_OSPF permit 20
</strong>
<strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#redistribute connected subnets route-map CONNECTED_TO_OSPF
</strong></code></pre>

The route-map above will deny 172.16.1.1 /32 and permits everything else. After attaching it to the redistribute command you’ll see this on R2 and R3:

<pre><code><strong>R2#show ip route ospf 
</strong>
      172.16.0.0/32 is subnetted, 2 subnets
O E2     172.16.2.1 [110/20] via 192.168.12.1, 00:00:03, FastEthernet0/0
O E2     172.16.3.1 [110/20] via 192.168.12.1, 00:00:03, FastEthernet0/0
</code></pre>

<pre><code><strong>R3#show ip route ospf
</strong>
      172.16.0.0/32 is subnetted, 2 subnets
O E2     172.16.2.1 [110/20] via 192.168.23.2, 00:00:07, FastEthernet0/0
O E2     172.16.3.1 [110/20] via 192.168.23.2, 00:00:07, FastEthernet0/0
O IA  192.168.12.0/24 [110/2] via 192.168.23.2, 00:20:34, FastEthernet0/0
</code></pre>

It’s gone from the routing table…mission accomplished! Let’s take a look at the final method…

### Summary No-Advertise

The last method to filter a type 5 LSA is a nice trick that you can do with the summary-address command. Let me show you how to use this to filter 172.16.2.1 /32:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#summary-address 172.16.2.1 255.255.255.255 not-advertise 
</strong></code></pre>

The trick is to add the **not-advertise** parameter to the summary-address command. Whatever matches the summary route will no longer be advertised:

<pre><code><strong>R2#show ip route ospf 
</strong>
      172.16.0.0/32 is subnetted, 1 subnets
O E2     172.16.3.1 [110/20] via 192.168.12.1, 00:01:40, FastEthernet0/0
</code></pre>

<pre><code><strong>R3#show ip route ospf
</strong>
      172.16.0.0/32 is subnetted, 1 subnets
O E2     172.16.3.1 [110/20] via 192.168.23.2, 00:01:44, FastEthernet0/0
O IA  192.168.12.0/24 [110/2] via 192.168.23.2, 00:22:11, FastEthernet0/0
</code></pre>

There we go, it’s gone from the routing tables!

## Conclusion

You have now seen three different methods how you can get rid of type 5 LSAs. Another method that prevents LSA type 5 from entering the area is using a [stub area](https://networklessons.com/ospf/introduction-to-ospf-stub-areas).

Be careful what filtering technique you use if you learn this for a CCIE R\&S lab. The devil is in the details…the distribute-list is actually filtering the network while the route-map and summary-address prevent the router from advertising something.

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
 ip address 172.16.0.1 255.255.255.255
!
interface Loopback1
 ip address 172.16.1.1 255.255.255.255
!
interface Loopback2
 ip address 172.16.2.1 255.255.255.255
!
interface Loopback3
 ip address 172.16.3.1 255.255.255.255
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
router ospf 1
 summary-address 172.16.2.1 255.255.255.255 not-advertise
 redistribute connected subnets route-map CONNECTED_TO_OSPF
 network 192.168.12.0 0.0.0.255 area 0
 distribute-list R1_L0 out
!
ip access-list standard R1_L0
 deny   172.16.0.1
 permit any
ip access-list standard R1_L1
 permit 172.16.1.1
!
route-map CONNECTED_TO_OSPF deny 10
 match ip address R1_L1
!
route-map CONNECTED_TO_OSPF permit 20
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
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 1
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
interface GigabitEthernet0/1
 ip address 192.168.23.3 255.255.255.0
!
router ospf 1
 network 192.168.23.0 0.0.0.255 area 1
!
end
```
{% endtab %}
{% endtabs %}

I hope this has been useful, if you have any questions just leave a comment!
