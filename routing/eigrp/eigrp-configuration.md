# EIGRP Configuration

EIGRP is one of the routing protocols you must master if you want to pass the Cisco CCNA exam. In this lesson, I’ll walk you through the configuration. If you have no idea what EIGRP is or how it works, you should read my [Introduction to EIGRP](https://networklessons.com/cisco/ccnp-encor-350-401/introduction-to-eigrp) first. This is the topology that we will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-ccna-topology.png" alt=""><figcaption></figcaption></figure>

In the topology above, I have four routers. All interfaces are FastEthernet with the exception of the link between R1 and R2. That’s where we use an Ethernet link. Behind R4, there is a loopback interface.

Let’s start by configuring EIGRP between R1 and R3:

<pre><code><strong>R1(config)#router eigrp 1
</strong><strong>R1(config-router)#no auto-summary 
</strong><strong>R1(config-router)#network 192.168.13.0
</strong></code></pre>

<pre><code><strong>R3(config)#router eigrp 1
</strong><strong>R3(config-router)#no auto-summary 
</strong><strong>R3(config-router)#network 192.168.13.0
</strong></code></pre>

Configuring EIGRP is similar to RIP. The “1” is the AS number, and it has to be the same on all routers! We require the no auto-summary command because by default EIGRP **behaves classful**, and we want it to be classless.

{% hint style="info" %}
`no auto-summary` is enabled by default since IOS 15.
{% endhint %}

After typing in these commands, this is what you will see:

```
R1#
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 192.168.13.3 (FastEthernet0/0) is up: new adjacency
```

```
R3#
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 192.168.13.1 (FastEthernet0/0) is up: new adjacency
```

Our routers have become EIGRP neighbors. We can also verify this with a command:

<pre><code><strong>R1#show ip eigrp neighbors 
</strong>IP-EIGRP neighbors for process 1
H   Address                Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                           (sec)         (ms)       Cnt Num
0   192.168.13.3            Fa0/0             12 00:11:58 1275  5000  0  3
</code></pre>

<pre><code><strong>R3#show ip eigrp neighbors 
</strong>IP-EIGRP neighbors for process 1
H   Address                Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                           (sec)         (ms)       Cnt Num
0   192.168.13.1            Fa0/0             14 00:11:47   15   200  0  3
</code></pre>

Use `show ip eigrp neighbors` to verify that we have a working EIGRP neighbor adjacency. This seems to be the case.

Let’s configure all the network commands, so all routers become EIGRP neighbors and advertise their networks:

<pre><code><strong>R1(config)#router eigrp 1
</strong><strong>R1(config-router)#network 192.168.12.0
</strong></code></pre>

<pre><code><strong>R3(config)#router eigrp 1
</strong><strong>R3(config-router)#network 192.168.34.0
</strong></code></pre>

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)#no auto-summary
</strong><strong>R2(config-router)#network 192.168.12.0
</strong><strong>R2(config-router)#network 192.168.24.0
</strong></code></pre>

<pre><code><strong>R4(config)#router eigrp 1
</strong><strong>R4(config-router)#no auto-summary
</strong><strong>R4(config-router)#network 192.168.24.0
</strong><strong>R4(config-router)#network 192.168.34.0
</strong><strong>R4(config-router)#network 4.0.0.0
</strong></code></pre>

These network commands will activate EIGRP on all interfaces and advertise all networks that we have. Let’s verify our work:

<pre><code><strong>R1#show ip eigrp neighbors 
</strong>IP-EIGRP neighbors for process 1
H   Address                Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                           (sec)         (ms)       Cnt Num
1   192.168.12.2            Et1/0             14 00:20:08   12   200  0  15
0   192.168.13.3            Fa0/0             11 00:43:34  428  2568  0  12
</code></pre>

<pre><code><strong>R2#show ip eigrp neighbors 
</strong>IP-EIGRP neighbors for process 1
H   Address                Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                           (sec)         (ms)       Cnt Num
1   192.168.24.4            Fa1/0             12 00:19:22   12   200  0  14
0   192.168.12.1            Et0/0             13 00:20:17   10   200  0  15
</code></pre>

<pre><code><strong>R3#show ip eigrp neighbors 
</strong>IP-EIGRP neighbors for process 1
H   Address                Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                           (sec)         (ms)       Cnt Num
1   192.168.34.4            Fa1/0             14 00:19:29    8   200  0  13
0   192.168.13.1            Fa0/0             12 00:43:53    9   200  0  14
</code></pre>

```
R4#show ip eigrp neighbors 
IP-EIGRP neighbors for process 1
H   Address                Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                           (sec)         (ms)       Cnt Num
1   192.168.34.3            Fa1/0             12 00:19:39  658  3948  0  13
0   192.168.24.2            Fa0/0             14 00:19:42  537  3222  0  16
```

Each router has two EIGRP neighbors, so this is looking good. Now let me show you the routing table of R1:

<pre><code><strong>R1#show ip route eigrp 
</strong>     4.0.0.0/24 is subnetted, 1 subnets
D       4.4.4.0 [90/158720] via 192.168.13.3, 00:21:48, FastEthernet0/0
D    192.168.24.0/24 [90/33280] via 192.168.13.3, 00:21:53, FastEthernet0/0
D    192.168.34.0/24 [90/30720] via 192.168.13.3, 00:21:50, FastEthernet0/0
</code></pre>

The first thing you might notice is that you see a “D” for the EIGRP entries. The reason that you see a “D” and not an “E” is that the last one is already taken for EGP, an old routing protocol that we don’t use anymore. “D” stands for “dual,” which is the mechanism behind EIGRP. Let’s take a closer look at one of these entries:

```
D       4.4.4.0 [90/158720] via 192.168.13.3, 00:21:48, FastEthernet0/0
```

What exactly do we see here?

* 4.4.4.0 is the network that we have learned.
* 90 is the administrative distance for EIGRP.
* 158720 is the metric. This is the feasible distance that you see.
* Via 192.168.13.3 is the next hop IP address. In this case, it’s R3.

The metric that EIGRP uses isn’t as easy as OSPF or RIP. The numbers to work with are very large.

Why did R1 decide to use the link through R3 to get to network 4.4.4.0 /24? The answer is in the EIGRP topology table:

<pre><code><strong>R1#show ip eigrp topology
</strong>IP-EIGRP Topology Table for AS(1)/ID(192.168.12.1)

Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
r - reply Status, s - sia Status

P 4.4.4.0/24, 1 successors, FD is 158720
via 192.168.13.3 (158720/156160), FastEthernet0/0
via 192.168.12.2 (412160/156160), Ethernet1/0
</code></pre>

Use `show ip eigrp topology` to see the EIGRP topology table. This is an important command so let me describe what we have here:

`P 4.4.4.0/24, 1 successors, FD is 158720`

The first line tells us that we have a successor. We already knew this because we saw it in the routing table. FD is the feasible distance to reach 4.4.4.0 /24.

```
via 192.168.13.3 (158720/156160), FastEthernet0/0
```

Above you see two important numbers:

* 158720 is the feasible distance.
* 156160 is the advertised distance.

via 192.168.12.2 (412160/156160), Ethernet1/0

Above, you see the information for the feasible successor (R2):

* 412160 is the feasible distance.
* 156160 is the advertised distance.

The reason that this is a feasible successor is because the advertised distance (156160) is lower than the feasible distance of the successor (158720)

So to summarize what we have seen:

* R1 knows how to reach network 4.4.4.0 /24.
* The path through R3 is the successor because the feasible distance is 158720.
* The path through R2 is the feasible successor because the advertised distance (156160) is lower than the feasible distance of the successor (158720).

Right now, only the successor is in the routing table, but we activate load balancing so that the feasible successor will also be used. This is how we do it:

<pre><code><strong>R1#show ip eigrp topology        
</strong>IP-EIGRP Topology Table for AS(1)/ID(192.168.12.1)

Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 4.4.4.0/24, 1 successors, FD is 158720
        via 192.168.13.3 (158720/156160), FastEthernet0/0
        via 192.168.12.2 (412160/156160), Ethernet1/0
</code></pre>

The feasible successor will be used if its feasible distance is lower than the feasible distance of the successor multiplied by a multiplier.

158720 x 3 (multiplier)= 476160

412160 (feasible successor) is lower than 476160, so we will use this feasible successor. Let’s try another multiplier:

<pre><code><strong>R1(config)#router eigrp 1
</strong><strong>R1(config-router)#variance 3
</strong></code></pre>

I need to use the `variance` command to set this multiplier. Let’s check our routing table now:

<pre><code><strong>R1#show ip route eigrp 
</strong>     4.0.0.0/24 is subnetted, 1 subnets
D       4.4.4.0 [90/158720] via 192.168.13.3, 00:00:37, FastEthernet0/0
                [90/412160] via 192.168.12.2, 00:00:37, Ethernet1/0
</code></pre>

Great! Now we see both entries in the routing table. You have now seen how EIGRP selects the successor, feasible successor and how to enable load balancing over feasible successors.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet0/0
 ip address 192.168.13.1 255.255.255.0
!
interface Ethernet1/0
 ip address 192.168.12.1 255.255.255.0
!
router eigrp 1
 no auto-summary
 network 192.168.13.0
 network 192.168.12.0
 variance 3
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface Ethernet0/0
 ip address 192.168.12.2 255.255.255.0
!
interface Fastethernet1/0
 ip address 192.168.24.2 255.255.255.0
!
router eigrp 1
 no auto-summary
 network 192.168.12.0
 network 192.168.24.0
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
interface FastEthernet0/0
 ip address 192.168.13.3 255.255.255.0
!
interface FastEthernet1/0
 ip address 192.168.34.3 255.255.255.0
!
router eigrp 1
 no auto-summary
 network 192.168.13.0
 network 192.168.34.0
!
end
```
{% endtab %}

{% tab title="R4" %}
```
hostname R4
!
interface FastEthernet0/0
 ip address 192.168.24.4 255.255.255.0
!
interface Fastethernet1/0
 ip address 192.168.34.4 255.255.255.0
!
interface Loopback0
 ip adress 4.4.4.4 255.255.255.0
!
router eigrp 1
 no auto-summary
 network 192.168.24.0
 network 192.168.34.0
 network 4.0.0.0
!
end
```
{% endtab %}
{% endtabs %}

That’s all I have for you. I hope this was useful to you and if you have any questions, please leave a comment!
