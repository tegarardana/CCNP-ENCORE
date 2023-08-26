# Static Route Configuration

In this lesson, we’ll take a look at static routes and in particular, how to configure them.

Let me show you the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/hq-branch-router.png" alt=""><figcaption></figcaption></figure>

Look at the network in the picture above. We have a network with two sites, headquarters, and a branch office.

The headquarters is connected to the Branch office. Behind the branch office is a network with the 2.2.2.0 /24 network. We want to ensure that the headquarters can reach the 2.2.2.0 /24 network.

Let me show you how we configure this network using a static route:

<pre><code><strong>HQ>enable
</strong><strong>HQ#configure terminal
</strong></code></pre>

First, I’ll go to enable mode and enter configuration mode.

<pre><code><strong>HQ(config)#interface FastEthernet 0/0
</strong><strong>HQ(config-if)#no shutdown
</strong><strong>HQ(config-if)#ip address 192.168.12.1 255.255.255.0
</strong></code></pre>

<pre><code><strong>Branch>enable
</strong><strong>Branch#configure terminal
</strong><strong>Branch(config)#interface fastEthernet0/0
</strong><strong>Branch(config-if)#no shutdown
</strong><strong>Branch(config-if)#ip address 192.168.12.2 255.255.255.0
</strong><strong>Branch(config-if)#exit
</strong><strong>Branch(config)#interface fastEthernet 1/0
</strong><strong>Branch(config-if)#no shutdown
</strong><strong>Branch(config-if)#ip address 2.2.2.2 255.255.255.0
</strong></code></pre>

Then I’ll configure the IP addresses on the interfaces; don’t forget to do a `no shutdown` on the interfaces.

Let’s take a look at the routing tables of both routers:

<pre><code><strong>HQ#show ip route 
</strong>Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, 
       ia - IS-IS inter area, * - candidate default,
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

C    192.168.12.0/24 is directly connected, FastEthernet0/0
</code></pre>

<pre><code><strong>Branch#show ip route 
</strong>Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1,
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

C    192.168.12.0/24 is directly connected, FastEthernet0/0
     2.0.0.0/24 is subnetted, 1 subnets
C       2.2.2.0 is directly connected, FastEthernet1/0
</code></pre>

Use the `show ip route` command to view the routing table. This is what a router uses to make decisions about where to forward IP packets to. By default, a router only knows its directly connected networks. We configured an IP address with a subnet mask on the interface, so the router also knows the network address.

* Router HQ knows about network 192.168.12.0/24.
* Router Branch knows about network 192.168.12.0/24 and 2.2.2.0/24.

At this moment our HQ router has no idea how to reach network 2.2.2.0/24 because there is no entry in the routing table. What will happen when we try to reach it? Let’s check:

<pre><code><strong>HQ#ping 2.2.2.2
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2.2.2.2, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
</code></pre>

The ping will fail. This router checks its routing table, and discovers that it doesn’t know how to reach network 2.2.2.0 /24, and will drop the traffic. Let’s use a static route to tell router HQ how to reach this network!

<pre><code><strong>HQ(config)#ip route 2.2.2.0 255.255.255.0 192.168.12.2
</strong></code></pre>

We use the ip route command to create a static route. Let me break it down for you:

* 2.2.2.0 is the network we want to reach.
* 255.255.255.0 is the subnet mask of the network.
* 192.168.12.2 is called the next hop IP address. It’s the IP address where we want to send traffic to. In this example, that’s the branch router.

I’m telling router HQ that it can reach network 2.2.2.0 /24 by sending traffic to IP address 192.168.12.2 (the Branch router).

Let’s take another look at the routing table to see if anything has changed:

<pre><code><strong>HQ#show ip route 
</strong>Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 -IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

C    192.168.12.0/24 is directly connected, FastEthernet1/0
     1.0.0.0/24 is subnetted, 1 subnets
C       1.2.3.0 is directly connected, FastEthernet0/0
     2.0.0.0/24 is subnetted, 1 subnets
S       2.2.2.0 [1/0] via 192.168.12.2
</code></pre>

We can now see an entry for network 2.2.2.0/24 in our routing table. Whenever router HQ has traffic for network 2.2.2.0 /24, it will send it to IP address 192.168.12.2 (router Branch). Let’s see if our ping is now working:

<pre><code><strong>HQ#ping 2.2.2.2
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2.2.2.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/12 ms
</code></pre>

Bingo, now it’s working. Router HQ knows how to reach network 2.2.2.0 /24 because of our static route.

Are you following me so far? Whenever an IP packet arrives at a router, it will check its routing table to see if it knows about the destination network. If it does, it will forward the IP packet, and if it has no idea where to send traffic, it will drop the IP packet.\


{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="HQ" %}
```
hostname HQ
!
interface FastEthernet 0/0
 ip address 192.168.12.1 255.255.255.0
!
ip route 2.2.2.0 255.255.255.0 192.168.12.2
!
end
```
{% endtab %}

{% tab title="Branch" %}
```
hostname Branch
!
interface fastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
interface fastEthernet 1/0
 ip address 2.2.2.2 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

There is another situation where a static route might be useful. Let me demonstrate another network:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/internet-isp-router-1.png" alt=""><figcaption></figcaption></figure>

In the picture above, our HQ router is connected to an ISP (Internet Service Provider). There are many networks on the Internet, so do we require all of those networks on the Internet in our routing table? The answer is no because we can use a default route. Let me show you what it is:

<pre><code><strong>HQ(config)#interface fastEthernet 1/0
</strong><strong>HQ(config-if)#ip address 1.2.3.2 255.255.255.0
</strong><strong>HQ(config-if)#no shutdown
</strong><strong>HQ(config-if)#exit
</strong></code></pre>

First, we’ll configure an IP address on the FastEthernet 1/0 of the HQ router.

<pre><code><strong>HQ#ping 1.2.3.1
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.2.3.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/12 ms
</code></pre>

It’s always a good idea to check connectivity. A quick ping to the ISP router proves that we can reach the ISP.

<pre><code><strong>HQ#show ip route 
</strong>Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 -IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

C       1.2.3.0 is directly connected, FastEthernet1/0
</code></pre>

Right now, the HQ router only knows how to reach network 1.2.3.0/24 because it’s directly connected. Let’s configure a default route so that we can reach the Internet:

<pre><code><strong>HQ(config)#ip route 0.0.0.0 0.0.0.0 1.2.3.1
</strong></code></pre>

Let me explain this one:

* The first 0.0.0.0 is the network address; in this case, it means all networks.
* The second 0.0.0.0 is the subnet mask; all zeroes means all subnet masks.
* 1.2.3.1 is the next hop IP address. In this case, the IP address of the ISP router.

In other words, this static route will match all networks, and that’s why we call it a default route. When our router doesn’t know where to deliver IP packets to, we’ll throw it over the fence towards the ISP and it will be their job to deliver it…sounds good, right?\


{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="HQ" %}
```
hostname HQ
!
interface FastEthernet 1/0
 ip address 1.2.3.2 255.255.255.0
!
ip route 0.0.0.0 0.0.0.0 1.2.3.1
!
end
```
{% endtab %}

{% tab title="ISP" %}
```
hostname ISP
!
interface FastEthernet 0/0
 ip address 1.2.3.1 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

\
It is important to know that routers will always use the most specific match in their routing table. Let me give you an example:

<pre><code><strong>Router#show ip route static 
</strong>     192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
S       192.168.1.0/24 [1/0] via 10.2.2.2
S       192.168.1.128/25 [1/0] via 10.3.3.2
S    192.168.0.0/16 [1/0] via 10.1.1.2
</code></pre>

Imagine the router above receives an IP packet with destination IP address 192.168.1.140. Will it send the IP packet towards 10.2.2.2, 10.3.3.2, or 10.1.1.2?

All three entries in the routing table match this destination IP address, but 192.168.1.128 /25 is the most specific entry in this case. The IP packets will be forwarded to 10.3.3.2.

Now you know how a router uses its routing table and how to configure a static route. Are there any disadvantages to static routes? Let me show you an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/4-routers-with-networks.png" alt=""><figcaption></figcaption></figure>

In the picture above, I have many routers and a lot of networks. If I want to configure full reachability between the routers, then I have to configure many static routes to make this work, and you don’t have any backups. If a link fails, you’ll need to edit your static route and send traffic in another direction. The picture above would be more suitable for dynamic routing.

I hope this lesson has been helpful to you. If you have any questions, please leave a comment.
