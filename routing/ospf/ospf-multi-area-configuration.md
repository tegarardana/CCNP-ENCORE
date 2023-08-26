# OSPF Multi Area Configuration

In my [introduction on OSPF](https://networklessons.com/cisco/ccnp-encor-350-401/introduction-to-ospf) I explained why we sometimes use multiple areas for OSPF and in our [configuration example](https://networklessons.com/cisco/ccnp-encor-350-401/basic-ospf-configuration), I showed you how to configure OSPF for a single area. This time, we’ll take a look how you can configure multi area OSPF.

We will use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/04/ospf-two-areas-multi-area.png" alt=""><figcaption></figcaption></figure>

Above we have R1 and R2 in area 0, the backbone area. Between R1 and R3, we will use area 1 and between R2/R4 we will use area 2. R3 and R4 have a loopback interface with an IP address that we will advertise in their area.

## Configuration

Let’s start with all network commands to get OSPF up and running. The network command defines to which area each interface will belong.First, we will configure R1 and R2 for the backbone area:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong></code></pre>

Let’s configure R1 and R3 for area 1:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 192.168.13.0 0.0.0.255 area 1
</strong></code></pre>

<pre><code><strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#network 192.168.13.0 0.0.0.255 area 1
</strong><strong>R3(config-router)#network 3.3.3.3 0.0.0.0 area 1
</strong></code></pre>

And last but not least, R2 and R4 for area 2:

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 192.168.24.0 0.0.0.255 area 2
</strong></code></pre>

<pre><code><strong>R4(config)#router ospf 1
</strong><strong>R4(config-router)#network 192.168.24.0 0.0.0.255 area 2
</strong><strong>R4(config-router)#network 4.4.4.4 0.0.0.0 area 2
</strong></code></pre>

Those are all the network commands we need.

## Verification

Let’s verify our work. First, let’s make sure we have OSPF neighbors:

<pre><code><strong>R1#show ip ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.24.2      1   FULL/DR         00:00:36    192.168.12.2    GigabitEthernet0/1
3.3.3.3           1   FULL/BDR        00:00:34    192.168.13.3    GigabitEthernet0/2
</code></pre>

R1 has formed a neighbor adjacency with R2 and R3. Let’s check R2:

<pre><code><strong>R2#show ip ospf neighbor
</strong>
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.13.1      1   FULL/BDR        00:00:34    192.168.13.1    GigabitEthernet0/1
4.4.4.4           1   FULL/BDR        00:00:30    192.168.24.4    GigabitEthernet0/2
</code></pre>

R2 has formed neighbor adjacencies with R1 and R4. The s**how ip** ospf **neighbor** command, however, doesn’t tell me anything about the areas that are used. If you want to see this, you could add the **detail** parameter like this:

<pre><code><strong>R2#show ip ospf neighbor detail 
</strong> Neighbor 192.168.13.1, interface address 192.168.12.1
    In the area 0 via interface GigabitEthernet0/1
    Neighbor priority is 1, State is FULL, 6 state changes
    DR is 192.168.12.2 BDR is 192.168.12.1
    Options is 0x12 in Hello (E-bit, L-bit)
    Options is 0x52 in DBD (E-bit, L-bit, O-bit)
    LLS Options is 0x1 (LR)
    Dead timer due in 00:00:33
    Neighbor is up for 00:17:30
    Index 1/1/1, retransmission queue length 0, number of retransmission 0
    First 0x0(0)/0x0(0)/0x0(0) Next 0x0(0)/0x0(0)/0x0(0)
    Last retransmission scan length is 0, maximum is 0
    Last retransmission scan time is 0 msec, maximum is 0 msec
 Neighbor 4.4.4.4, interface address 192.168.24.4
    In the area 2 via interface GigabitEthernet0/2
    Neighbor priority is 1, State is FULL, 6 state changes
    DR is 192.168.24.2 BDR is 192.168.24.4
    Options is 0x12 in Hello (E-bit, L-bit)
    Options is 0x52 in DBD (E-bit, L-bit, O-bit)
    LLS Options is 0x1 (LR)
    Dead timer due in 00:00:31
    Neighbor is up for 00:15:57
    Index 1/1/2, retransmission queue length 0, number of retransmission 0
    First 0x0(0)/0x0(0)/0x0(0) Next 0x0(0)/0x0(0)/0x0(0)
    Last retransmission scan length is 0, maximum is 0
    Last retransmission scan time is 0 msec, maximum is 0 msec
</code></pre>

Above you can see that interface GigabitEthernet0/1 is in area 0 and interface GigabitEthernet0/2 is in area 2. Another good command to find area information is **show ip protocols**:

<pre><code><strong>R2#show ip protocols 
</strong>*** IP Routing is NSF aware ***

Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 192.168.24.2
  It is an area border router
  Number of areas in this router is 2. 2 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.12.0 0.0.0.255 area 0
    192.168.24.0 0.0.0.255 area 2
  Routing Information Sources:
    Gateway         Distance      Last Update
    4.4.4.4              110      00:16:04
    192.168.13.1         110      00:16:53
  Distance: (default is 110)
</code></pre>

Above you can see which networks belong to which area:

* Network 192.168.12.0 in area 0.
* Network 192.168.24.0 in area 2.

Let’s check our routing tables. Let’s start with R1:

<pre><code><strong>R1#show ip route ospf
</strong>Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/2] via 192.168.13.3, 00:01:47, GigabitEthernet0/2
      4.0.0.0/32 is subnetted, 1 subnets
O IA     4.4.4.4 [110/3] via 192.168.12.2, 00:00:54, GigabitEthernet0/1
O IA  192.168.24.0/24 [110/2] via 192.168.12.2, 00:01:44, GigabitEthernet0/1
</code></pre>

Above we see three OSPF entries. The first one is for 3.3.3.3/32, the loopback interface of R3. It shows up with an O since this is an intra-area route. R1 has also learned about 4.4.4.4/32 and 192.168.24.0/24. These two entries show up as O IA since they are inter-area routes.

R2 has a similar output:

<pre><code><strong>R2#show ip route ospf
</strong>
      3.0.0.0/32 is subnetted, 1 subnets
O IA     3.3.3.3 [110/3] via 192.168.12.1, 00:02:19, GigabitEthernet0/1
      4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/2] via 192.168.24.4, 00:01:29, GigabitEthernet0/2
O IA  192.168.13.0/24 [110/2] via 192.168.12.1, 00:02:24, GigabitEthernet0/1
</code></pre>

Above we see that R2 has learned about 3.3.3.3/32 and 192.168.13.0/24 which area inter-area routes. 4.4.4.4/32 is an intra-area route.

Let’s check R3:

<pre><code><strong>R3#show ip route ospf
</strong>
      4.0.0.0/32 is subnetted, 1 subnets
O IA     4.4.4.4 [110/4] via 192.168.13.1, 00:01:57, GigabitEthernet0/1
O IA  192.168.12.0/24 [110/2] via 192.168.13.1, 00:02:50, GigabitEthernet0/1
O IA  192.168.24.0/24 [110/3] via 192.168.13.1, 00:02:47, GigabitEthernet0/1
</code></pre>

Everything that R3 has learned is from another area, that’s why we only see inter-area routes here. The same thing applies to R4:

<pre><code><strong>R4#show ip route ospf
</strong>
      3.0.0.0/32 is subnetted, 1 subnets
O IA     3.3.3.3 [110/4] via 192.168.24.2, 00:02:13, GigabitEthernet0/1
O IA  192.168.12.0/24 [110/2] via 192.168.24.2, 00:02:13, GigabitEthernet0/1
O IA  192.168.13.0/24 [110/3] via 192.168.24.2, 00:02:13, GigabitEthernet0/1
</code></pre>

Just to be sure, let’s try a quick ping between R3 and R4 to prove that our multi-area OSPF configuration is working:

<pre><code><strong>R3#ping 4.4.4.4 source 3.3.3.3
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 4.4.4.4, timeout is 2 seconds:
Packet sent with a source address of 3.3.3.3 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 9/11/13 ms
</code></pre>

Our ping is successful. That will be all for now.\


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
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 192.168.13.1 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.13.0 0.0.0.255 area 1
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
 ip address 192.168.24.2 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.24.0 0.0.0.255 area 2
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
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
!
interface GigabitEthernet0/1
 ip address 192.168.13.3 255.255.255.0
!
router ospf 1
 network 3.3.3.3 0.0.0.0 area 1
 network 192.168.13.0 0.0.0.255 area 1
!
end
```
{% endtab %}

{% tab title="R4" %}
```
hostname R4
!
ip cef
!
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
!
interface GigabitEthernet0/1
 ip address 192.168.24.4 255.255.255.0
!
router ospf 1
 network 4.4.4.4 0.0.0.0 area 2
 network 192.168.24.0 0.0.0.255 area 2
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

You have now learned how to configure multiple OSPF areas and how to verify OSPF routes in the routing table that are from different areas.
