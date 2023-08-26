# EIGRP Variance

This lesson demonstrates how to use the EIGRP variance command to load balance EIGRP over feasible successors. If you have no idea what I’m talking about then it’s best to read my lesson about [EIGRP unequal load balancing ](https://networklessons.com/cisco/ccnp-encor-350-401/eigrp-unequal-cost-load-balancing)first before you continue.

To demonstrate EIGRP load balancing, I will use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/06/eigrp-variance-example-topology.png" alt=""><figcaption></figcaption></figure>

The routers above are all running EIGRP. R1 is connected to R2, R3, and R4 using a _FastEthernet, Ethernet, and Serial link_. On the right side, you see R5 which has a loopback interface that is configured with network 5.5.5.5 /32. We will enable EIGRP on all interfaces and take a look at what path R1 will choose when we want to reach 5.5.5.5 /32.

Let’s enable EIGRP on all routers using the “shotgun approach”:

<pre><code>R1,R2,R3,R4 &#x26; R5:
<strong>(config)#router eigrp 1
</strong><strong>(config-router)#no auto-summary
</strong><strong>(config-router)#network 0.0.0.0
</strong></code></pre>

If you are configuring this yourself, make sure you check that all routers have formed EIGRP neighbor adjacencies before you continue.

Let’s take a look to see what path R1 will choose to reach 5.5.5.5 /32:

<pre><code><strong>R1#show ip route | begin 5.5.5.5
</strong>D       5.5.5.5 [90/158720] via 192.168.12.2, 00:00:04, FastEthernet0/0
</code></pre>

It will take the path through R2 which makes sense since this is the FastEthernet interface. If you want to see more detailed information, you can use the following command:

<pre><code><strong>R1#show ip route 5.5.5.5
</strong>Routing entry for 5.5.5.5/32
  Known via "eigrp 1", distance 90, metric 158720, type internal
  Redistributing via eigrp 1
  Last update from 192.168.12.2 on FastEthernet0/0, 00:00:07 ago
  Routing Descriptor Blocks:
  * 192.168.12.2, from 192.168.12.2, 00:00:07 ago, via FastEthernet0/0
      Route metric is 158720, traffic share count is 1
      Total delay is 5200 microseconds, minimum bandwidth is 100000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2
</code></pre>

To see exactly why R1 has selected R2 as the successor for this network, we’ll have to take a look at the EIGRP topology table:

<pre><code><strong>R1#show ip eigrp topology 5.5.5.5 255.255.255.255
</strong>IP-EIGRP (AS 1): Topology entry for 5.5.5.5/32
  State is Passive, Query origin flag is 1, 1 Successor(s), FD is 158720
  Routing Descriptor Blocks:
  192.168.12.2 (FastEthernet0/0), from 192.168.12.2, Send flag is 0x0
      Composite metric is (158720/156160), Route is Internal
      Vector metric:
        Minimum bandwidth is 100000 Kbit
        Total delay is 5200 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
  192.168.14.4 (Serial2/0), from 192.168.14.4, Send flag is 0x0
      Composite metric is (2300416/156160), Route is Internal
      Vector metric:
        Minimum bandwidth is 1544 Kbit
        Total delay is 25100 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
  192.168.13.3 (Ethernet1/0), from 192.168.13.3, Send flag is 0x0
      Composite metric is (412160/156160), Route is Internal
      Vector metric:
        Minimum bandwidth is 10000 Kbit
        Total delay is 6100 microseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 2
</code></pre>

Above you see the different values for the feasible distance and advertised distance. The lowest feasible distance is **158.720,** and it’s the path through R2 which makes it the **successor**.

R3 and R4 have been selected as **feasible successors** because their advertised distance of **156.160** is lower than the feasible distance (158.720) of R2.

So far, so good, we found the successor, and we know that R3 and R4 are feasible successors. If we want to enable load balancing, we have to use the following formula:

```
FD of feasible successor < FD of successor * multiplier
```

So the feasible distance of the feasible successor has to be lower than the feasible distance of the successor that is multiplied with some value. Let’s look at an example, so that this makes more sense. Let’s say we want to load balance over R3:

* Feasible Distance of R2 (successor) = 158720
* Feasible Distance of R3 (feasible successor) =412160

```
412160 / 158720 = 2.59
```

So if I set the multiplier at something higher than 2.59, then R3 will be used for load balancing. The multiplier is configured using the **variance** command:

<pre><code><strong>R1(config)#router eigrp 1
</strong><strong>R1(config-router)#variance 3
</strong></code></pre>

Let’s take a look at R1 to see if this has any effect:

<pre><code><strong>R1#show ip route | begin 5.5.5.5
</strong>D 5.5.5.5 [90/412160] via 192.168.13.3, 00:00:42, Ethernet1/0
          [90/158720] via 192.168.12.2, 00:00:42, FastEthernet0/0
</code></pre>

Above, you can see that R1 has installed the path through R3 as well. EIGRP does “unequal” cost load balancing, and to see how it shares traffic among the interfaces, we have to use another command:

<pre><code><strong>R1#show ip route 5.5.5.5
</strong>Routing entry for 5.5.5.5/32
  Known via "eigrp 1", distance 90, metric 158720, type internal
  Redistributing via eigrp 1
  Last update from 192.168.13.3 on Ethernet1/0, 00:00:40 ago
  Routing Descriptor Blocks:
  * 192.168.13.3, from 192.168.13.3, 00:00:40 ago, via Ethernet1/0
      Route metric is 412160, traffic share count is 23
      Total delay is 6100 microseconds, minimum bandwidth is 10000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2
    192.168.12.2, from 192.168.12.2, 00:00:40 ago, via FastEthernet0/0
      Route metric is 158720, traffic share count is 60
      Total delay is 5200 microseconds, minimum bandwidth is 100000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2
</code></pre>

As you can see, EIGRP is sharing traffic in a 60:23 proportion, which means the FastEthernet link is used about 2.6 more often than the Ethernet link. What if we also want to use the serial link for load balancing? The feasible distance of R4 (2300416) is quite high. What kind of multiplier do we require to enable this link?

```
2300416 / 158720 = 14.49
```

When we set the multiplier to something higher than 14.49, we will use this link for load balancing. Let’s configure this using the variance command:

<pre><code><strong>R1(config)#router eigrp 1
</strong><strong>R1(config-router)#variance 15
</strong></code></pre>

Let’s verify our work:

<pre><code><strong>R1#show ip route | begin 5.5.5.5
</strong>D       5.5.5.5 [90/2300416] via 192.168.14.4, 00:00:05, Serial2/0
                [90/412160] via 192.168.13.3, 00:00:05, Ethernet1/0
                [90/158720] via 192.168.12.2, 00:00:05, FastEthernet0/0
</code></pre>

To see how EIGRP load balances traffic, we’ll take a detailed look again:

<pre><code><strong>R1#show ip route 5.5.5.5
</strong>Routing entry for 5.5.5.5/32
  Known via "eigrp 1", distance 90, metric 158720, type internal
  Redistributing via eigrp 1
  Last update from 192.168.13.3 on Ethernet1/0, 00:00:19 ago
  Routing Descriptor Blocks:
    192.168.14.4, from 192.168.14.4, 00:00:19 ago, via Serial2/0
      Route metric is 2300416, traffic share count is 17
      Total delay is 25100 microseconds, minimum bandwidth is 1544 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2
    192.168.13.3, from 192.168.13.3, 00:00:19 ago, via Ethernet1/0
      Route metric is 412160, traffic share count is 92
      Total delay is 6100 microseconds, minimum bandwidth is 10000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2
  * 192.168.12.2, from 192.168.12.2, 00:00:19 ago, via FastEthernet0/0
      Route metric is 158720, traffic share count is 240
      Total delay is 5200 microseconds, minimum bandwidth is 100000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 2
</code></pre>

Above, you can see that the FastEthernet link is used for most of the traffic, and the Serial link won’t be used much.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
```
{% endtab %}

{% tab title="R2" %}
```
```
{% endtab %}

{% tab title="R3" %}
```
```
{% endtab %}

{% tab title="R4" %}
```
```
{% endtab %}

{% tab title="R5" %}
```
```
{% endtab %}
{% endtabs %}

Hopefully, this lesson has helped you to understand the EIGRP variance command and unequal cost load balancing. Feel free to leave a comment if you have any questions.
