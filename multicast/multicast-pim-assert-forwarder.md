# Multicast PIM Assert (Forwarder)

Multicast PIM Assert is one of those important multicast topics that most people don’t really think about. Let’s take a look at the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/03/multicast-assert.png" alt=""><figcaption></figcaption></figure>

Above, you see four routers that are configured to use multicast. R1 is the source, and R4 is our receiver. As you can see, R2, R3, and R4 are connected to the same switch.

Now when R1 starts streaming multicast traffic towards R2 and R3, they will both forward multicast packets to R4, resulting in duplicate traffic. To stop this, PIM will **elect one PIM forwarder** for this segment. PIM doesn’t have any routing information itself but relies on other routing protocols that are configured, it will use this information to select the best forwarding path with the PIM assert mechanism.

{% hint style="info" %}
Don’t confuse the PIM forwarder with the [PIM DR (Designated Router)](https://networklessons.com/cisco/ccnp-encor-350-401/multicast-pim-designated-router). those are two different things!
{% endhint %}

When R2 and R3 both forward multicast packets to the 192.168.234.0 /24 segment, they will see each other’s multicast traffic, this will trigger the PIM assert mechanism. We will elect a PIM forwarder based on the following rules:

1. The router with the **lowest administrative distance** to the source of the multicast stream will be the elected PIM forwarder. This only happens if you use two routing protocols or when you use a static route pointing to the source.
2. If the AD is equal, we will compare the **unicast routing metric** toward the source.
3. If the AD and metric are both the same, we will elect the PIM forwarded based on the **highest IP address**.

The elected PIM forwarder will keep forwarding traffic to the receiver, while the loser will prune its interface.

Now let’s take a look at this in action! First, I’ll configure a basic PIM Dense mode setup:

<pre><code><strong>R1(config)#ip multicast-routing 
</strong>
<strong>R1(config)#interface loopback 0
</strong><strong>R1(config-if)#ip address 1.1.1.1 255.255.255.255
</strong><strong>R1(config-if)#ip pim dense-mode 
</strong>
<strong>R1(config)#interface fastEthernet 0/0
</strong><strong>R1(config-if)#ip pim dense-mode
</strong>
<strong>R1(config)#interface fastEthernet 0/1
</strong><strong>R1(config-if)#ip pim dense-mode
</strong></code></pre>

I will use a loopback interface on R1 as the source for our multicast stream.

<pre><code><strong>R2(config)#ip multicast-routing
</strong>
<strong>R2(config)#interface fastEthernet 0/0
</strong><strong>R2(config-if)#ip pim dense-mode
</strong>
<strong>R2(config)#interface fastEthernet 0/1
</strong><strong>R2(config-if)#ip pim dense-mode
</strong></code></pre>

<pre><code><strong>R3(config)#ip multicast-routing
</strong>
<strong>R3(config)#interface fastEthernet 0/0
</strong><strong>R3(config-if)#ip pim dense-mode
</strong>
<strong>R3(config)#interface fastEthernet 0/1
</strong><strong>R3(config-if)#ip pim dense-mode
</strong></code></pre>

R2 and R3 are simple. Just enable multicast routing and PIM Dense mode on the interfaces. Only R4 left:

<pre><code><strong>R4(config)#ip multicast-routing 
</strong>
<strong>R4(config)#interface fastEthernet 0/0
</strong><strong>R4(config-if)#ip pim dense-mode 
</strong><strong>R4(config-if)#ip igmp join-group 239.1.1.1
</strong></code></pre>

On R4, we will also enable PIM dense mode and make it listen to the 239.1.1.1 multicast group address.

I applied OSPF to all routers, using the quick “shotgun approach” to advertise everything:

<pre><code><strong>router ospf 1
</strong><strong> network 0.0.0.0 255.255.255.255 area 0
</strong></code></pre>

Now let’s start a multicast stream from R1:

<pre><code><strong>R1#ping 239.1.1.1 source loopback 0 repeat 9999
</strong>
Type escape sequence to abort.
Sending 9999, 100-byte ICMP Echos to 239.1.1.1, timeout is 2 seconds:
Packet sent with a source address of 1.1.1.1 

Reply to request 0 from 192.168.234.4, 12 ms
</code></pre>

You will see replies from R4. Now let’s check who is forwarding this traffic:

<pre><code><strong>R2#show ip mroute 1.1.1.1 239.1.1.1
</strong>IP Multicast Routing Table
Flags: D - Dense, S - Sparse, B - Bidir Group, s - SSM Group, C - Connected,
       L - Local, P - Pruned, R - RP-bit set, F - Register flag,
       T - SPT-bit set, J - Join SPT, M - MSDP created entry,
       X - Proxy Join Timer Running, A - Candidate for MSDP Advertisement,
       U - URD, I - Received Source Specific Host Report,
       Z - Multicast Tunnel, z - MDT-data group sender,
       Y - Joined MDT-data group, y - Sending to MDT-data group
Outgoing interface flags: H - Hardware switched, A - Assert winner
 Timers: Uptime/Expires
 Interface state: Interface, Next-Hop or VCD, State/Mode

(1.1.1.1, 239.1.1.1), 00:02:56/00:00:07, flags: PT
  Incoming interface: FastEthernet0/0, RPF nbr 192.168.12.1
  Outgoing interface list:
    FastEthernet0/1, Prune/Dense, 00:02:56/00:00:03
</code></pre>

<pre><code><strong>R3#show ip mroute 1.1.1.1 239.1.1.1  
</strong>IP Multicast Routing Table
Flags: D - Dense, S - Sparse, B - Bidir Group, s - SSM Group, C - Connected,
       L - Local, P - Pruned, R - RP-bit set, F - Register flag,
       T - SPT-bit set, J - Join SPT, M - MSDP created entry,
       X - Proxy Join Timer Running, A - Candidate for MSDP Advertisement,
       U - URD, I - Received Source Specific Host Report,
       Z - Multicast Tunnel, z - MDT-data group sender,
       Y - Joined MDT-data group, y - Sending to MDT-data group
Outgoing interface flags: H - Hardware switched, A - Assert winner
 Timers: Uptime/Expires
 Interface state: Interface, Next-Hop or VCD, State/Mode

(1.1.1.1, 239.1.1.1), 00:02:48/00:02:57, flags: T
  Incoming interface: FastEthernet0/0, RPF nbr 192.168.13.1
  Outgoing interface list:
    FastEthernet0/1, Forward/Dense, 00:02:48/00:00:00, A
</code></pre>

As you can see above, R2 has pruned its outgoing interface while R3 is forwarding it. Note that you also see the “A” indicating that R3 is the assert winner. R3 has won this PIM assert election because it has the highest IP address. Let’s change the AD on R2 to see if it becomes the new assert winner:

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#distance 100 0.0.0.0 255.255.255.255 1 
</strong></code></pre>

<pre><code><strong>R2(config)#access-list 1 permit host 1.1.1.1
</strong></code></pre>

We will change the AD on R2 to 100 for all prefixes matching access-list 1. Access-list 1 matches 1.1.1.1, which is the source of our multicast stream.

{% hint style="warning" %}
Keep in mind that when we work with multicast, we are concerned about the source IP addresses, not destination IP addresses like with unicast routing.
{% endhint %}

Let’s verify our work:

<pre><code><strong>R2#show ip route | include 1.1.1.1
</strong>O       1.1.1.1 [100/11] via 192.168.12.1, 00:01:32, FastEthernet0/0
</code></pre>

<pre><code><strong>R3#show ip route | include 1.1.1.1
</strong>O       1.1.1.1 [110/11] via 192.168.13.1, 00:01:20, FastEthernet0/0
</code></pre>

R2 now has a better administrative distance, so it should win the PIM assert election; let’s see if this is true:

<pre><code><strong>R2#show ip mroute 1.1.1.1 239.1.1.1
</strong>IP Multicast Routing Table
Flags: D - Dense, S - Sparse, B - Bidir Group, s - SSM Group, C - Connected,
L - Local, P - Pruned, R - RP-bit set, F - Register flag,
T - SPT-bit set, J - Join SPT, M - MSDP created entry,
X - Proxy Join Timer Running, A - Candidate for MSDP Advertisement,
U - URD, I - Received Source Specific Host Report,
Z - Multicast Tunnel, z - MDT-data group sender,
Y - Joined MDT-data group, y - Sending to MDT-data group
Outgoing interface flags: H - Hardware switched, A - Assert winner
Timers: Uptime/Expires
Interface state: Interface, Next-Hop or VCD, State/Mode

(1.1.1.1, 239.1.1.1), 00:00:11/00:02:53, flags: T
Incoming interface: FastEthernet0/0, RPF nbr 192.168.12.1
Outgoing interface list:
<strong>FastEthernet0/1, Forward/Dense, 00:00:11/00:00:00, A
</strong></code></pre>

There we have it. The “A” tells us that R2 is now the PIM assert winner. Let’s remove the lowered AD to play with the metric:

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#no distance 100 0.0.0.0 255.255.255.255 1
</strong></code></pre>

This will make R3 the forwarder again. We will now increase the cost on the FastEthernet0/0 link of R3 so that its metric to the source is worse than R2:

<pre><code><strong>R2#show ip ospf interface fa0/0 | include Cost
</strong>  Process ID 1, Router ID 192.168.234.2, Network Type BROADCAST, Cost: 10
</code></pre>

<pre><code><strong>R3#show ip ospf interface fa0/0 | include Cost
</strong>  Process ID 1, Router ID 192.168.234.3, Network Type BROADCAST, Cost: 10
</code></pre>

The default cost of the FastEthernet links is 10 on R2 and R3. Let’s increase it on R3:

<pre><code><strong>R3(config)#interface fastEthernet 0/0
</strong><strong>R3(config-if)#ip ospf cost 11
</strong></code></pre>

This is now the metric for R2 and R3 to reach the loopback0 interface of R1:

<pre><code><strong>R2#show ip route ospf | include 1.1.1.1
</strong>O       1.1.1.1 [110/11] via 192.168.12.1, 00:06:58, FastEthernet0/0
</code></pre>

<pre><code><strong>R3#show ip route ospf | include 1.1.1.1
</strong>O       1.1.1.1 [110/12] via 192.168.13.1, 00:00:44, FastEthernet0/0
</code></pre>

R2 has a lower metric to reach the source of the multicast stream so it will be the PIM assert winner:

<pre><code><strong>R2#show ip mroute 1.1.1.1 239.1.1.1 | begin FastEthernet0/1
</strong><strong>    FastEthernet0/1, Forward/Dense, 00:01:41/00:00:00, A
</strong></code></pre>

And you can see this in the output above. If you want to see what it looks like behind the scenes, you can also enable a debug:

<pre><code><strong>R2#debug ip pim
</strong>PIM debugging is on
</code></pre>

You will see something like this:

```
Send v2 Assert on FastEthernet0/1 for 239.1.1.1, source 1.1.1.1, metric [110/11] 
PIM(0): Assert metric to source 1.1.1.1 is [110/11]
PIM(0): We win, our metric [110/11]
PIM(0): (1.1.1.1/32, 239.1.1.1) oif FastEthernet0/1 in Forward state
PIM(0): Received v2 Assert on FastEthernet0/1 from 192.168.234.3
PIM(0): Assert metric to source 1.1.1.1 is [110/12]
PIM(0): We win, our metric [110/11]
PIM(0): (1.1.1.1/32, 239.1.1.1) oif FastEthernet0/1 in Forward state
```

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
ip multicast-routing
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 ip pim dense-mode
!
interface FastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
 ip pim dense-mode
!
interface FastEthernet0/1
 ip address 192.168.13.1 255.255.255.0
 ip pim dense-mode
!
router ospf 1
 network 0.0.0.0 255.255.255.255 area 0
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
ip multicast-routing
!
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
 ip pim dense-mode
!
interface FastEthernet0/1
 ip address 192.168.234.2 255.255.255.0
 ip pim dense-mode
!
router ospf 1
 network 0.0.0.0 255.255.255.255 area 0
!
access-list 1 permit 1.1.1.1
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
ip multicast-routing
!
interface FastEthernet0/0
 ip address 192.168.13.3 255.255.255.0
 ip pim dense-mode
 ip ospf cost 11
!
interface FastEthernet0/1
 ip address 192.168.234.3 255.255.255.0
 ip pim dense-mode
!
router ospf 1
 network 0.0.0.0 255.255.255.255 area 0
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
ip multicast-routing
!
interface FastEthernet0/0
 ip address 192.168.234.4 255.255.255.0
 ip pim dense-mode
 ip igmp join-group 239.1.1.1
!
router ospf 1
 network 0.0.0.0 255.255.255.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

I hope this explanation and walkthrough help you understand PIM assert. If you have any questions feel free to leave a comment!
