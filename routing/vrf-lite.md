# VRF Lite

In this lesson you will learn about **VRFs (Virtual Routing and Forwarding)**. By default a router uses a **single global routing table** that contains all the directly connected networks and prefixes that it learned through static or dynamic routing protocols.

VRFs are like VLANs for routers, instead of using a single global routing table we can use multiple virtual routing tables. Each interface of the router is assigned to a different VRF.

VRFs are commonly used for MPLS deployments, when we use VRFs without MPLS then we call it **VRF lite**. That’s what we will focus on in this lesson. Let’s take a look at an example topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/isp-router-customer-red-blue.png" alt=""><figcaption></figcaption></figure>

In the topology above we have one ISP router and two customers called “Red” and “Blue”. Each customer has two sites and those are connected to the ISP router. The ISP router has only one global routing table so if we connect everything like the topology above, this is what the routing table will look like:

<pre><code><strong>ISP#show ip route connected
</strong>C    192.168.4.0/24 is directly connected, FastEthernet3/0
C    192.168.1.0/24 is directly connected, FastEthernet0/0
C    192.168.2.0/24 is directly connected, FastEthernet1/0
C    192.168.3.0/24 is directly connected, FastEthernet2/0
</code></pre>

The ISP router has a single global routing table that has all 4 directly connected networks. Let’s use VRFs to change this, I want to create a seperate routing table for customer “Blue” and “Red”. First we have to create these VRFs:

<pre><code><strong>ISP(config)#ip vrf Red
</strong><strong>ISP(config-vrf)#exit
</strong><strong>ISP(config)#ip vrf Blue
</strong><strong>ISP(config-vrf)#exit
</strong></code></pre>

Globally we create the VRFs, one for each customer. Our next step is to add the interfaces of the ISP router to the correct VRF. Here’s how:

<pre><code><strong>ISP(config)#interface FastEthernet 0/0
</strong><strong>ISP(config-if)#ip vrf forwarding Blue
</strong>% Interface FastEthernet0/0 IP address 192.168.1.254 removed due to enabling VRF Blue
<strong>ISP(config-if)#ip address 192.168.1.254 255.255.255.0
</strong></code></pre>

On the interface level we use the **ip vrf forwarding command** to assign the interface to the correct VRF. Once you do this , you’ll have to add the IP address again. Let’s configure the remaining interfaces:

<pre><code><strong>ISP(config)#interface FastEthernet 1/0
</strong><strong>ISP(config-if)#ip vrf forwarding Red
</strong><strong>ISP(config-if)#ip address 192.168.2.254 255.255.255.0
</strong>
<strong>ISP(config)#interface FastEthernet 2/0
</strong><strong>ISP(config-if)#ip vrf forwarding Blue
</strong><strong>ISP(config-if)#ip address 192.168.3.254 255.255.255.0
</strong>
<strong>ISP(config)#interface FastEthernet 3/0
</strong><strong>ISP(config-if)#ip vrf forwarding Red
</strong><strong>ISP(config-if)#ip address 192.168.4.254 255.255.255.0
</strong></code></pre>

All interfaces are now configured. There’s a useful command you can use to see all the VRFs and their interfaces:

<pre><code><strong>ISP#show ip vrf
</strong>  Name                             Default RD          Interfaces
  Blue                                                 Fa0/0
                                                       Fa2/0
  Red                                                  Fa1/0
                                                       Fa3/0
</code></pre>

Our VRFs are configured, let’s take a look at the global routing table of the ISP router:

<pre><code><strong>ISP#show ip route connected
</strong>
</code></pre>

The global routing table has no entries, this is because all interfaces were added to a VRF. Let’s check the VRF routing tables:

<pre><code><strong>ISP#show ip route vrf Blue connected
</strong>C    192.168.1.0/24 is directly connected, FastEthernet0/0
C    192.168.3.0/24 is directly connected, FastEthernet2/0
</code></pre>

<pre><code><strong>ISP#show ip route vrf Red connected
</strong>C    192.168.4.0/24 is directly connected, FastEthernet3/0
C    192.168.2.0/24 is directly connected, FastEthernet1/0
</code></pre>

We use the show ip route command but you’ll need to specify which VRF you want to look at. As you can see, each VRF has its own routing table with the interfaces that we configured earlier.

If you want to do something on the router like sending a ping then you’ll have to specify which VRF you want to use. By default it will use the global routing table. Here’s an example how to send a ping:

<pre><code><strong>ISP#ping vrf Blue 192.168.1.1
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
</code></pre>

That’s easy enough, just don’t forget to specify the correct VRF. The same thing applies to routing (protocols). For example if you want to configure a static route you’ll have to specify the correct VRF. Take a look at the example below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/vrf-static-route-loopback.png" alt=""><figcaption></figcaption></figure>

Router Blue1 has a loopback interface with IP address 1.1.1.1 /32. Let’s create a static route on the ISP router so that we can reach it:

<pre><code><strong>ISP(config)#ip route vrf Blue 1.1.1.1 255.255.255.255 192.168.1.1
</strong></code></pre>

We use the same ip route command but I specified to what VRF the static route belongs. Let’s see if this works:

<pre><code><strong>ISP#ping vrf Blue 1.1.1.1
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/24/52 ms
</code></pre>

Easy enough, the ping works. What about routing protocols? We can use OSPF, EIGRP, BGP…no problem at all. Let’s look at an example for OSPF:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/03/vrf-ospf-customer-blue-red.png" alt=""><figcaption></figcaption></figure>

Customer “Blue” and “Red” both want to use OSPF to advertise their networks. Since we use VRFs, everything is seperated. Let’s start with the OSPF configuration for customer Blue:

<pre><code><strong>Blue1(config)#router ospf 1
</strong><strong>Blue1(config-router)#network 192.168.1.0 0.0.0.255 area 0
</strong><strong>Blue1(config-router)#network 1.1.1.1 0.0.0.0 area 0
</strong></code></pre>

<pre><code><strong>Blue2(config)#router ospf 1
</strong><strong>Blue2(config-router)#network 192.168.3.0 0.0.0.255 area 0
</strong><strong>Blue2(config-router)#network 3.3.3.3 0.0.0.0 area 0
</strong></code></pre>

The OSPF configuration for the customer routers is pretty straight-forward. On the ISP router, we’ll have to specify what VRF we want to use:

<pre><code><strong>ISP(config)#router ospf 1 vrf Blue
</strong><strong>ISP(config-router)#network 192.168.1.0 0.0.0.255 area 0
</strong><strong>ISP(config-router)#network 192.168.3.0 0.0.0.255 area 0
</strong></code></pre>

We configure OSPF process 1 and specify the VRF that we want to use, that’s all there is to it. Let’s do the same for customer Red:

<pre><code><strong>Red1(config)#router ospf 1
</strong><strong>Red1(config-router)#network 192.168.2.0 0.0.0.255 area 0
</strong><strong>Red1(config-router)#network 2.2.2.2 0.0.0.0 area 0
</strong></code></pre>

<pre><code><strong>Red2(config)#router ospf 1
</strong><strong>Red2(config-router)#network 192.168.4.0 0.0.0.255 area 0
</strong><strong>Red2(config-router)#network 4.4.4.4 0.0.0.0 area 0
</strong></code></pre>

<pre><code><strong>ISP(config)#router ospf 2 vrf Red
</strong><strong>ISP(config-router)#network 192.168.2.0 0.0.0.255 area 0
</strong><strong>ISP(config-router)#network 192.168.4.0 0.0.0.255 area 0
</strong></code></pre>

The configuration is similar, I had to use another process ID on the ISP router since the first one is used for customer Blue. Here’s what the VRF routing tables on the ISP router look like now:

<pre><code><strong>ISP#show ip route vrf Blue ospf
</strong>
Routing Table: Blue

     1.0.0.0/32 is subnetted, 1 subnets
O       1.1.1.1 [110/2] via 192.168.1.1, 00:00:24, FastEthernet0/0
     3.0.0.0/32 is subnetted, 1 subnets
O       3.3.3.3 [110/2] via 192.168.3.3, 00:00:24, FastEthernet2/0
</code></pre>

<pre><code><strong>ISP#show ip route vrf Red ospf
</strong>
Routing Table: Red

     2.0.0.0/32 is subnetted, 1 subnets
O       2.2.2.2 [110/2] via 192.168.2.2, 00:00:19, FastEthernet1/0
     4.0.0.0/32 is subnetted, 1 subnets
O       4.4.4.4 [110/2] via 192.168.4.4, 00:00:19, FastEthernet3/0
</code></pre>

Two seperate routing tables with the prefixes from each VRF, this is looking good.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="Blue1" %}
```
hostname Blue1
!
ip cef
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.1.1 255.255.255.0
!
router ospf 1
 network 1.1.1.1 0.0.0.0 area 0
 network 192.168.1.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="Blue2" %}
```
hostname Blue2
!
ip cef
!
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.3.3 255.255.255.0
!
router ospf 1
 network 3.3.3.3 0.0.0.0 area 0
 network 192.168.3.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="Red1" %}
```
hostname Red1
!
ip cef
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.2.2 255.255.255.0
!
router ospf 1
 network 2.2.2.2 0.0.0.0 area 0
 network 192.168.2.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="Red2" %}
```
hostname Red2
!
ip cef
!
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
!
interface FastEthernet0/0
 ip address 192.168.4.4 255.255.255.0
!
router ospf 1
 network 4.4.4.4 0.0.0.0 area 0
 network 192.168.4.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="ISP" %}
```
hostname ISP
!
ip cef
!
ip vrf Blue
!
ip vrf Red
!
interface FastEthernet0/0
 ip vrf forwarding Blue
 ip address 192.168.1.254 255.255.255.0
!
interface FastEthernet1/0
 ip vrf forwarding Red
 ip address 192.168.2.254 255.255.255.0
!
interface FastEtherne2/0
 ip vrf forwarding Blue
 ip address 192.168.3.254 255.255.255.0
!
interface FastEthernet3/0
 ip vrf forwarding Red
 ip address 192.168.4.254 255.255.255.0
!
router ospf 1 vrf Blue
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.3.0 0.0.0.255 area 0
!
router ospf 2 vrf Red
 network 192.168.2.0 0.0.0.255 area 0
 network 192.168.4.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

\
This is what VRF lite is about, it has one downside though…it’s not a scalable solution. In our example we only used a single ISP router but what if we want to use VRFs and multiple ISP routers? That’s something we’ll discuss in the [EVN (Easy Virtual Network) lesson](https://networklessons.com/tag/vrf/cisco-evn-easy-virtual-network).

If you have any questions, feel free to leave a comment!
