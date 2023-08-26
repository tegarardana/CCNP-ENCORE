# EIGRP K Values Formula

EIGRP uses different K values to determine the best path to each destination:

* **K1**
* **K2**
* **K3**
* **K4**
* **K5**

These K values are only numbers to **scale numbers** in the metric calculation. The formula we use for the metric calculation looks like this:

{% hint style="info" %}
$$Metric = [K1*bandwidth + ((K2*bandwidth)/(256-load))+K3*delay]$$
{% endhint %}

If K5 is not equal to 0:

{% hint style="info" %}
$$Metric = Metric*[K5/(reliability+K4)]$$
{% endhint %}

If you look at the formula, you can see that the **bandwidth, load, delay, and reliability** influence the metric. We can see what K values are enabled or disabled by default:

<pre><code><strong>Router#show ip protocols 
</strong>Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP metric weight K1=1, K2=0, K3=1, K4=0, K5=0
  EIGRP maximum hopcount 100
  EIGRP maximum metric variance 1
  Redistributing: eigrp 1
  EIGRP NSF-aware route hold timer is 240s
  Automatic network summarization is not in effect
  Maximum path: 4
  Routing for Networks:
    1.1.1.0/24
    192.168.12.0
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.12.2          90      00:14:30
  Distance: internal 90 external 170
</code></pre>

In this example where I used the `show ip protocols` command, you can see which K-values are enabled by default. Only K1 and K3 are enabled by default.

Let’s walk through the different metric components to see what they are:

Bandwidth:

<pre><code><strong>Router#show interfaces fastEthernet 0/0
</strong>FastEthernet0/0 is up, line protocol is up 
  Hardware is AmdFE, address is cc02.58a9.0000 (bia cc02.58a9.0000)
  Internet address is 192.168.12.1/24
  MTU 1500 bytes, BW 100000 Kbit, DLY 100 usec,
</code></pre>

If you use the show interface FastEthernet 0/0 command you can see the interface information. The example above only shows part of the output. You can see the bandwidth is 100000 Kbit which is a 100Mbit interface. We can change the bandwidth of an interface:

<pre><code><strong>Router(config)#interface fa0/0
</strong><strong>Router(config-if)#bandwidth ?
</strong>  &#x3C;1-10000000>  Bandwidth in kilobits
  inherit       Specify that bandwidth is inherited
  receive       Specify receive-side bandwidth

<strong>Router(config-if)#bandwidth 500
</strong></code></pre>

Bandwidth is a static value which can be changed by using the bandwidth command. Keep in mind this doesn’t change the actual bandwidth of the interface! This command is ONLY used to influence routing protocols like EIGRP. It’s not like you can slow down electric signals through a wire…if you want to limit the traffic on an interface you’ll need QoS (Quality of Service) which is a story for another day!

<pre><code><strong>Router#show interfaces fastEthernet 0/0
</strong>FastEthernet0/0 is up, line protocol is up 
  Hardware is AmdFE, address is cc02.58a9.0000 (bia cc02.58a9.0000)
  Internet address is 192.168.12.1/24
  MTU 1500 bytes, BW 500 Kbit, DLY 100 usec,
</code></pre>

Here you see the result of changing the bandwidth on the interface. Something to remember is that EIGRP will use the lowest bandwidth in the path from A to B (since this is the bottleneck).

Load:

<pre><code><strong>Router#show interfaces fastEthernet 0/0
</strong>FastEthernet0/0 is up, line protocol is up 
  Hardware is AmdFE, address is cc02.58a9.0000 (bia cc02.58a9.0000)
  Internet address is 192.168.12.1/24
  MTU 1500 bytes, BW 500 Kbit, DLY 100 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
</code></pre>

The load will show you how busy the interface is based on the packet rate and the bandwidth on the interface. This is a value that can change over time so it’s a dynamic value.

Delay:

<pre><code><strong>Router#show interfaces fastEthernet 0/0
</strong>FastEthernet0/0 is up, line protocol is up 
  Hardware is AmdFE, address is cc02.58a9.0000 (bia cc02.58a9.0000)
  Internet address is 192.168.12.1/24
  MTU 1500 bytes, BW 500 Kbit, DLY 100 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
</code></pre>

Delay reflects the time it will take for packets to cross the link and is a static value. Cisco IOS will have default delay values for the different types of interface. A FastEthernet interface has a default delay of 100 usec.

<pre><code><strong>Router(config)#interface fa0/0
</strong><strong>Router(config-if)#delay ?
</strong>  &#x3C;1-16777215>  Throughput delay (tens of microseconds)

<strong>Router(config-if)#delay 50
</strong></code></pre>

If you use the delay command you can change this value to influence routing protocols like EIGRP. It doesn’t actually change the delay for this interface but it is only used to influence routing protocols.

<pre><code><strong>Router#show interfaces fastEthernet 0/0
</strong>FastEthernet0/0 is up, line protocol is up 
  Hardware is AmdFE, address is cc02.58a9.0000 (bia cc02.58a9.0000)
  Internet address is 192.168.12.1/24
  MTU 1500 bytes, BW 500 Kbit, DLY 500 usec,
</code></pre>

Above you see the delay that I changed. EIGRP will accumulate all the delay values in the path from A to B.

Reliability:

<pre><code><strong>Router#show interfaces fastEthernet 0/0
</strong>FastEthernet0/0 is up, line protocol is up 
  Hardware is AmdFE, address is cc02.58a9.0000 (bia cc02.58a9.0000)
  Internet address is 192.168.12.1/24
  MTU 1500 bytes, BW 500 Kbit, DLY 500 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
</code></pre>

Reliability at 255/255 is 100%. This means that you don’t have issues on the physical or data-link layer. If you are having issues this value will decrease. Since this is something that can change it’s a dynamic value.

MTU:

```
Router#show interfaces fastEthernet 0/0
FastEthernet0/0 is up, line protocol is up 
  Hardware is AmdFE, address is cc02.58a9.0000 (bia cc02.58a9.0000)
  Internet address is 192.168.12.1/24
  MTU 1500 bytes, BW 500 Kbit, DLY 500 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
```

MTU or Maximum Transmission Unit is being exchanged between EIGRP neighbors but not used for the metric calculation.

By default, **only K1 and K3 are enabled** and we don’t use K2 or K4. This means that only bandwidth and delay are used in the formula.

Why not? Because loading and reliability are dynamic values and they can change over time. You don’t want your EIGRP routers calculating 24/7 and sending updates to each other just because the load or reliability of an interface has changed. We want routing protocols to be nice and quiet and only base their routing decisions on static values like bandwidth and delay. If you only use those two static values our EIGRP routers don’t have to do any recalculation unless an interface goes down or a router died.

Since only K1 and K3 are enabled we can simplify the EIGRP formula:

{% hint style="info" %}
**Metric = bandwidth (slowest link) + delay (sum of delays)**
{% endhint %}

* Bandwidth: \[10⁷ / minimum bandwidth in the path] \* 256.
* Delay: sums of delays in the path multiplied by 256 (in tens of microseconds).

So the formula looks like:

{% hint style="info" %}
**Metric = (10**⁷ **/ minimum bandwidth) \* 256 + (sum of delays) \* 256**
{% endhint %}

The multiplication of 256 is done so EIGRP is compatible with IGRP (the predecessor of EIGRP).

Let me show you an example so we can break down this formula:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/07/eigrp-t1-links-bandwidth-delay.png" alt=""><figcaption></figcaption></figure>

We are looking at R1 and calculating the distance to get to R5. As you can see there is an upper path with some T1 interfaces and a 64kbps link. The path below has two 256kbps links.

A T1 interface has a bandwidth of 1.554Mbit which is obviously better than 256kbps but the bottleneck in the upper path is our 64kbps link. Let’s throw these numbers for the upper path in the EIGRP metric formula and see what happens:

The lowest bandwidth in the upper path is our 64kbps link so the EIGRP bandwidth calculation will look like this:

Bandwidth = (107 / slowest link) \* 256\
Bandwidth = (10,000,000 / 64) \* 256 = 156,250 \* 256 = 40,000,000

Now let’s look at the delay calculation for the upper path:

Delay = \[sum of delays] \* 256\
Delay = \[1000+1000+1000] \* 256\
Delay = 768,000

Let’s add those numbers together and we’ll have the total metric:

Metric = bandwidth + delay\
Metric = 40,000,000 + 768,000\
Metric = 40,768,000

Having fun yet? Let’s do the lower path as well!

The lowest bandwidth in the lower path is 256kbps link so the EIGRP bandwidth calculation will look like this:

Bandwidth = (107 / slowest link) \* 256\
Bandwidth = (10,000,000 / 256) \* 256 = 39062.5 \* 256 = 10,000,000

Now let’s look at the delay calculation for the lower path:

Delay = \[sum of delays] \* 256\
Delay = \[1000+1000] \* 256\
Delay = 512,000

Let’s add those numbers together and we’ll have the total metric:

Metric = bandwidth + delay\
Metric = 10,000,000 + 512,000\
Metric = 10,512,000

Upper path metric = 40,768,000\
Lower path metric = 10,512,000

The lower metric will be installed as the successor route in the routing table so the lower path will be used in this example.

Phew! Does this make your head spin? I think you and I both agree we should let EIGRP do the metric calculations and not us (that’s why we have routers right!). The important lesson I wanted to show you here is that EIGRP uses the slowest bandwidth in the path and the sum of delays. You don’t have to know this formula by heart but understand it. No need to do any manual calculations on the exam!

The metrics in EIGRP are a pain to work with since the values are so LARGE! If you want to practice with EIGRP you can try to disable all the K-values except K3. This will make EIGRP only use delay as the metric. The metric values will be much lower and easier to work with since you don’t have to think about the lowest bandwidth in the path. I like to do this when I’m teaching people how to calculate feasible successors and configure EIGRP load balancing.

Hopefully, this lesson has helped you to understand EIGRP K-values! If you have any questions, feel free to leave a comment.
