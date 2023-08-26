# EIGRP Wide Metrics

EIGRP uses the bandwidth, delay, reliability, load and K values to calculate a composite cost metric. The problem with this metric is that it doesn’t scale for high bandwidth interfaces. The composite cost metric is calculated like this:

```
EIGRP composite cost metric = 256*((K1*Scaled Bandwidth) + (K2*Scaled Bandwidth)/(256 – Load) + (K3*Scaled Delay)*(K5/(Reliability + K4)))
```

In the formula above, you can see that the bandwidth is scaled. It’s done with the following formula:

```
Scaled bandwidth = (107/minimum bandwidth (Bw) in kilobits per second)
```

The delay is also scaled, using this formula:

```
Scaled Delay = (Delay/10)
```

By default, only K1 and K3 are enabled, only bandwidth and delay are used. This means the composite cost metric can be simplified to the following formula:

```
EIGRP composite cost metric = 256*(Scaled Bandwidth + Scaled Delay)
```

The scaled bandwidth formula is unable to differentiate between anything faster than 10 GigabitEthernet. The lowest delay that we can configure is 10 microseconds, that’s the delay GigabitEthernet interfaces offer. Anything above GigabitEthernet will also have a delay of 10 microseconds.

Here are some examples to help you understand this:

* GigabitEthernet:
  * Scaled bandwidth: 10000000 / 1000000 = 10
  * Scaled delay: 10 / 10 = 1
  * Composite metric: 10 + 1 \* 256 = 2816
* 10 GigabitEthernet:
  * Scaled bandwidth: 10000000 / 10000000 = 1
  * Scaled delay: 10 / 10 = 1
  * Composite metric: 1 + 1 \* 256 = 512
* 11 GigabitEthernet:
  * Scaled bandwidth: 10000000 / 11000000 = 0.9 (rounded to 0)
  * Scaled delay: 10 / 10 = 1
  * Composite metric: 0 + 1 \* 256 = 256
* 20 GigabitEthernet:
  * Scaled bandwidth: 10000000 / 20000000 = 0.5 (rounded to 0)
  * Scaled delay: 10 / 10 = 1
  * Composite metric: 0 + 1 \* 256 = 256
* 40 GigabitEthernet:
  * Scaled bandwidth: 10000000 / 40000000 = 0.25 (rounded to 0)
  * Scaled delay: 10 / 10 = 1
  * Composite metric: 0 + 1 \* 256 = 256

Above, you can see that the scaled bandwidth of anything higher than 10 GigabitEthernet has a value that is below one and is rounded down to zero. The delay has a value of one for GigabitEthernet or faster interfaces.

Any interface above 10 GigabitEthernet will always have a composite cost metric of 256. To EIGRP, there is no difference between anything above 10 GigabitEthernet or 40 GigabitEthernet interfaces and this might cause undesirable equal-cost load balancing in your network.

The metric and its formulas as explained above is called the **EIGRP classic metrics**. To solve this problem, EIGRP supports **wide metrics** which uses **64-bit** values instead of the 32-bit values that EIGRP classics metric uses. EIGRP wide metrics supports interfaces up to 4.2 terabits and uses a different composite cost metric formula. It has the following components:

* Throughput: this is the bandwidth, it uses a new scaled bandwidth formula.
* Latency: this is the delay, in picoseconds. 1000000000 picoseconds = 1 millisecond. It also uses a new latency scaling formula.
* Reliability: the same as with EIGRP classic metrics.
* Load: the same as with EIGRP classic metrics.
* MTU: the same as with EIGRP classic metrics.
* Hop Count: the same as with EIGRP classic metrics.
* Extended Metrics: these are currently not used but reserved for future extensions. There are three extended metrics as of this moment:
  * Jitter
  * Energy
  * Quiescent Energy

To add these extended metrics to the composite metric, a new K value was introduced called **K6**.

Let’s take a look at the wide metrics composite metric formula:

```
EIGRP Composite Cost Metric = [(K1*Minimum Throughput + {K2*Minimum Throughput} / 256-Load) + (K3*Total Latency) + (K6*Extended Attributes)]* [K5/(K4 + Reliability)]
```

In the formula above, you can see the new K6 value. By default only throughput and latency are used so we can simplify the formula:

```
EIGRP Composite Cost Metric = (K1*Minimum Throughput) + (K3*Total Latency)
```

The minimum throughput uses the following formula:

```
Minimum Throughput = (107 * 65536)/Bw)
```

The formula for the total latency depends on the interface bandwidth. For interfaces below GigabitEthernet, this is the formula:

```
Total Latency = (Delay * 65536) / 10
```

For interfaces above GigabitEthernet, we use this formula:

```
Total Latency = (107 * 65536/10) / Bandwidth
```

The wide metrics composite cost metric with its 64-bit values does introduce a new problem. The routing table on Cisco IOS only supports 32-bit values so when we install a metric in the routing table, it will be scaled down.

EIGRP wide metrics is only supported for [EIGRP named mode](https://networklessons.com/eigrp/eigrp-named-mode-configuration/) and activated automatically; you can’t enable or disable it. EIGRP routers will automatically detect if their neighbors support wide metrics. If so, wide metrics are automatically used, otherwise classic metrics are used. If you have mixed neighbors on a multi-access, then both metric formats will be added.

Once you have configured EIGRP named mode, you can verify that wide metrics are used like this:

<pre><code><strong>R1#show ip protocols | begin eigrp
</strong>Routing Protocol is "eigrp 12"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 VR(MY_NAME) Address-Family Protocol for AS(12)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0 K6=0
    Metric rib-scale 128
    Metric version 64bit
    Soft SIA disabled
    NSF-aware route hold timer is 240
    Router-ID: 192.168.12.1
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
      Total Prefix Count: 1
      Total Redist Count: 0
</code></pre>

In the output above, we see:

* The metric weight includes K6 which tells us that we are using wide metrics.
* The metric rib scale with a value of 128. The EIGRP metric will be divided by this number so that we have a metric that fits in the routing table.
* The metric version which is 64-bit. This also tells us that we are using wide metrics.

## Conclusion

In this lesson, you have learned how EIGRP classic metrics are limited since there is no differentiation of the metric for high bandwidth interfaces. EIGRP wide metric solves this by using a 64-bit value and a new composite metric formula that supports up to 4.2 terabit interfaces. You have also seen how you can verify that EIGRP wide metrics are used on your router.
