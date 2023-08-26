# Introduction to Administrative Distance (AD)

Administrative distance is one of those routing concepts that most CCNA students have difficulty with understanding. In this short lesson, I’ll explain to you what administrative distance is and how it works.

Let me show you an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/administrative-distance.png" alt=""><figcaption></figcaption></figure>

Imagine we have a network that is running **two routing protocols at the same time**, OSPF and EIGRP. Both routing protocols give information to R1.

* EIGRP tells us the router should send IP packets using the path on the top.
* OSPF tells us the router should send IP packets using the path on the bottom.

What routing information are we going to use? Both? Use OSPF or EIGRP?

The answer is that when two routing protocols are giving us information about the same destination network, we have to make a choice…you can’t go left and right at the same time. We need to look at the administrative distance or AD.

Let me show you the administrative distance list:

<table><thead><tr><th width="196"></th><th width="387">Administrative Distance</th></tr></thead><tbody><tr><td>Directly connected</td><td>0</td></tr><tr><td>Static route</td><td>1</td></tr><tr><td>EIGRP</td><td>90</td></tr><tr><td>OSPF</td><td>110</td></tr><tr><td>RIP</td><td>120</td></tr></tbody></table>

The lower the administrative distance, the better. As you can see, a directly connected route has an AD of 0. This makes sense since there’s nothing better than having it directly connected to your router. A static route has a very low administrative distance of 1, which also makes sense since this is something you configure manually. Sometimes you use a static route to “overrule” a routing protocol’s decisions.

EIGRP has an administrative distance of 90, which makes sense since it’s a Cisco routing protocol. OSPF has 110, and RIP has 120. In our example above, we will use the information EIGRP tells us in the routing table since its AD of 90 is better (lower) than OSPF, which has 110.

Let’s look at an example of an actual router. This is the topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/r1-r2-r3-rip-static.png" alt=""><figcaption></figcaption></figure>

Above we see that R1 is connected to both R2 and R3. Here’s the routing table:

<pre><code><strong>R1#show ip route
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

      2.0.0.0/24 is subnetted, 1 subnets
R        2.2.2.0 [120/1] via 192.168.12.2, 00:00:21, GigabitEthernet0/1
      3.0.0.0/24 is subnetted, 1 subnets
S        3.3.3.0 [1/0] via 192.168.13.3
      192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, GigabitEthernet0/1
L        192.168.12.1/32 is directly connected, GigabitEthernet0/1
      192.168.13.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.13.0/24 is directly connected, GigabitEthernet0/2
L        192.168.13.1/32 is directly connected, GigabitEthernet0/2
</code></pre>

Above you can see that R1 has learned 2.2.2.0 /24 through RIP. Between the brackets, we find:

```
[120/1]
```

120 is the administrative distance, and 1 is the metric. In the case of RIP, that’s the hop count.

R1 also has a static route for 3.3.3.0 /24 to R3. Between the brackets, we find:

```
[1/0]
```

1 is the administrative distance. Since this is a static route, there is no metric, so it’s 0. I hope this has helped to understand the administrative distance.
