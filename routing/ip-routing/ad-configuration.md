# AD Configuration

When two or more sources are giving you information about a certain prefix you need to choose which information you are going to use. For example, OSPF might tell you to go “left” if you want to reach network X, and EIGRP might tell you that you need to go “right”. Who do you trust? OSPF or EIGRP? You can’t put both in the routing table for network X.

The **administrative distance** solves this problem. When two sources give us information about the **exact same network** we’ll have to make a decision and it’s done by looking at the administrative distance. Let me show you the different values:

<table><thead><tr><th width="234">Source</th><th width="216">Administrative Distance</th></tr></thead><tbody><tr><td>Directly connected</td><td>0</td></tr><tr><td>Static route</td><td>1</td></tr><tr><td>EIGRP summary</td><td>5</td></tr><tr><td>External BGP</td><td>20</td></tr><tr><td>EIGRP</td><td>90</td></tr><tr><td>IGRP</td><td>100</td></tr><tr><td>OSPF</td><td>110</td></tr><tr><td>IS-IS</td><td>115</td></tr><tr><td>RIP</td><td>120</td></tr><tr><td>ODR</td><td>160</td></tr><tr><td>External EIGRP</td><td>170</td></tr><tr><td>Internal BGP</td><td>200</td></tr><tr><td>Unknown</td><td>255</td></tr></tbody></table>

The lower, the better…as you can see EIGRP has a lower administrative distance (90) than OSPF (110), so we will use EIGRP in my example.

Keep in mind:

* The administrative distance is only local and can be different for each router.
* The administrative distance can be **modified**.

Especially when we use redistribution, we sometimes have to change the administrative distance. Let me show you how you can do this:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#distance eigrp 90 160
</strong></code></pre>

Above we have EIGRP, and with the **`distance`** command, I can change the administrative distance for EIGRP globally. Internal EIGRP will keep its AD of 90, but external EIGRP will have an AD of 160. You will see this change in the routing table:

<pre><code><strong>R1#show ip route eigrp 
</strong>     3.0.0.0/24 is subnetted, 1 subnets
D EX    3.3.3.0 [160/1734656] via 192.168.12.2, 00:00:30, FastEthernet0/0
D EX 192.168.23.0/24 [160/1734656] via 192.168.12.2, 00:00:30,FastEthernet0/0
</code></pre>

You can verify it by looking at the routing table, the external networks on router R1 now have an AD of 160.

We can change the AD of the other routing protocols as well. Here are some examples:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#distance ospf external 150 inter-area 80 intra-area 80
</strong></code></pre>

For OSPF, you can change the external, inter-area, and intra-area administrative distance. In my example, I’ve set the external distance (type 5 and 7 external LSAs) to 150. Inter-area distance is 80, and intra-area is 80. This means that your router will now prefer OSPF information above EIGRP (AD 90).

The downside of the two examples above is that it applies to all prefixes. I can also change the administrative distance only for certain prefixes. Here’s how to do it:

<pre><code><strong>R1(config)#router rip
</strong><strong>R1(config-router)#distance 70 0.0.0.0 255.255.255.255 MY_PREFIXES
</strong></code></pre>

<pre><code><strong>R1(config)#ip access-list standard MY_PREFIXES
</strong><strong>R1(config-std-nacl)#permit 1.1.1.0 0.0.0.255
</strong></code></pre>

I use the **distance** command and combine it with a standard access-list called “MY\_PREFIXES”. All networks that match this access-list will have their AD changed to 70. Let’s check the routing table:

<pre><code><strong>R1#show ip route rip 
</strong>R    192.168.12.0/24 [120/10] via 192.168.23.2, 00:00:15, FastEthernet0/0
     1.0.0.0/24 is subnetted, 1 subnets
R       1.1.1.0 [70/10] via 192.168.23.2, 00:00:15, FastEthernet0/0
</code></pre>

Above, you see the new administrative distance for network 1.1.1.0 /24.

That’s all I wanted to show you for now. If you have any questions feel free to ask!
