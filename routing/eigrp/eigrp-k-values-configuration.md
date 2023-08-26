# EIGRP K Values Configuration

EIGRP is able to use the bandwidth, delay, load, reliability and MTU as input for its metric calculation. The metric is calculated using some weighting constants called the K values.

By default only bandwidth and delay are used for the metric calculation and Cisco recommends not to use load and reliability. The bandwidth and delay are both static values while load and reliability are dynamic, they change all the time.

For labs it might be useful to disable bandwidth as it simplifies the metric calculation. By default, EIGRP will use the lowest bandwidth in the path while the delays of all interfaces in the path are accumulated.

The [formula ](https://networklessons.com/cisco/ccnp-encor-350-401/eigrp-k-values-formula)to calculate the metric is quite complicated as I [explained here](https://networklessons.com/cisco/ccnp-encor-350-401/eigrp-k-values-formula). In this lesson we’ll take a look how we can configure these K values. This is the topology I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/r1-r2-fast-ethernet.png" alt=""><figcaption></figcaption></figure>

Let’s start by enabling EIGRP:

<pre><code>R1 &#x26; R2#
<strong>(config)#router eigrp 12
</strong><strong>(config-router)#network 192.168.12.0
</strong></code></pre>

With the default configuration EIGRP will use these K values:

<pre><code><strong>R1#show ip protocols | include K   
</strong>  EIGRP metric weight K1=1, K2=0, K3=1, K4=0, K5=0
</code></pre>

K1 and K3 are set to one, with these two enabled the routers will include bandwidth and delay in the metric calculation. Let’s change the K values so that our router will only use K3:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#metric weights ? 
</strong>  &#x3C;0-8>  Type Of Service (Only TOS 0 supported)
</code></pre>

We use the **metric weights** command to change the K values. The first value is for the TOS byte but as you can see it only supports a value of 0. The next values are for the actual K values:

<pre><code><strong>R1(config-router)#metric weights 0 ?
</strong>  &#x3C;0-255>  K1
</code></pre>

This one is for K1, let’s set it to zero. If you increase the weight here then bandwidth will be more of an influence for the metric calculation. Let’s continue:

<pre><code><strong>R1(config-router)#metric weights 0 0 ?
</strong>  &#x3C;0-255>  K2
</code></pre>

K2 is disabled by default so let’s keep it that way:

<pre><code><strong>R1(config-router)#metric weights 0 0 0 ?
</strong>  &#x3C;0-255>  K3
</code></pre>

K3 is enabled by default, let’s keep it enabled:

<pre><code><strong>R1(config-router)#metric weights 0 0 0 1 ?
</strong>  &#x3C;0-255>  K4
</code></pre>

K4 is also disabled by default, let’s keep it disabled. Last but not least is K5:

<pre><code><strong>R1(config-router)#metric weights 0 0 0 1 0 ?
</strong>  &#x3C;0-255>  K5
</code></pre>

This one refers to the MTU which isn’t really used in the metric calculation. Our final command will be:

<pre><code><strong>R1(config-router)#metric weights 0 0 0 1 0 0
</strong></code></pre>

As soon as you change the weight values you will see this on your console:

```
R1#
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 12: Neighbor 192.168.12.2 (FastEthernet0/0) is down: metric changed
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 12: Neighbor 192.168.12.2 (FastEthernet0/0) is down: K-value mismatch
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 12: Neighbor 192.168.12.2 (FastEthernet0/0) is down: Interface Goodbye received
```

The EIGRP neighbor adjacency is resetted since the K values changed. R2 is still using the default setting so you will get a notification about the K values mismatch. Let’s configure R2 as well:

<pre><code><strong>R2(config)#router eigrp 12
</strong><strong>R2(config-router)#metric weights 0 0 0 1 0 0
</strong></code></pre>

That’s it, our routers will become EIGRP neighbors again:

<pre><code><strong>R1#show ip eigrp neighbors 
</strong>IP-EIGRP neighbors for process 12
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
0   192.168.12.2            Fa0/0             13 00:10:36   75   450  0  6
</code></pre>

<pre><code><strong>R1#show ip protocols | include K
</strong>  EIGRP metric weight K1=0, K2=0, K3=1, K4=0, K5=0
</code></pre>

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet0/0
 bandwidth 500
 ip address 192.168.12.1 255.255.255.0
 delay 50
 duplex auto
 speed auto
 media-type rj45
!         
router eigrp 12
 metric weights 0 0 0 1 0 0
 network 192.168.12.0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
router eigrp 12
 metric weights 0 0 0 1 0 0
 network 192.168.12.0
!
end
```
{% endtab %}
{% endtabs %}

That’s all there is to it, this is how you configure the EIGRP K values. I hope this has been useful, if you have any questions feel free to leave a comment!
