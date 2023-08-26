# OSPF Reference Bandwidth

OSPF uses a simple formula to calculate the OSPF cost for an interface with this formula:

```
cost = reference bandwidth / interface bandwidth
```

The reference bandwidth is a value in Mbps that we can set ourselves. By default, this is 100Mbps on Cisco IOS routers. The interface bandwidth is something we can look up.

Let’s take a look at an example of how this works. I’ll use this router:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/cisco-router-fastethernet-serial-interface.png" alt=""><figcaption></figcaption></figure>

The router above has two interfaces, a FastEthernet and a serial interface:

<pre><code><strong>R1#show ip interface brief
</strong>Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.1.1     YES manual up                    up
Serial0/0                  192.168.2.1     YES manual up                    up
</code></pre>

Let’s enable OSPF on these interfaces:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 192.168.1.0 0.0.0.255 area 0
</strong><strong>R1(config-router)#network 192.168.2.0 0.0.0.255 area 0
</strong></code></pre>

After enabling OSPF, we can check what the reference bandwidth is:

<pre><code><strong>Router#show ip ospf | include Reference
</strong> Reference bandwidth unit is 100 mbps
</code></pre>

By default, this is 100 Mbps. Let’s see what cost values OSPF has calculated for our two interfaces:

<pre><code><strong>Router#show interfaces FastEthernet 0/0 | include BW
</strong>  MTU 1500 bytes, BW 100000 Kbit/sec, DLY 100 usec
</code></pre>

<pre><code><strong>Router#show ip ospf interface FastEthernet 0/0 | include Cost
</strong>  Process ID 1, Router ID 192.168.1.1, Network Type BROADCAST, Cost: 1
</code></pre>

The FastEthernet interface has a bandwidth of 100,000 kbps (100 Mbps), and the OSPF cost is 1. The formula to calculate the cost looks like this:

```
100.000 kbps reference bandwidth / 100.000 interface bandwidth = 1
```

What about the serial interface? Let’s find out:

<pre><code><strong>R1#show interfaces Serial 0/0 | include BW
</strong>  MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec,
</code></pre>

<pre><code><strong>R1#show ip ospf interface Serial 0/0 | include Cost 
</strong>  Process ID 1, Router ID 192.168.2.1, Network Type POINT_TO_POINT, Cost: 64
</code></pre>

The serial interface has a bandwidth of 1.544 kbps (1.5 Mbps) and a cost of 64. It was calculated like this:

```
100,000 kbps reference bandwidth / 1,544 kbps interface bandwidth = 64.76
```

It was rounded down to 64.

The default reference bandwidth of 100 Mbps can cause issues if you use Gigabit or 10 Gigabit interfaces. The lowest possible cost value is one so with the default reference bandwidth, a FastEthernet, Gigabit, and 10 Gigabit interface would have an OSPF cost of 1.

If you use Gigabit interfaces (or 10 Gigabit), then it’s better to change the reference bandwidth. You can do it like this:

<pre><code><strong>Router(config-router)#auto-cost reference-bandwidth ?
</strong>  &#x3C;1-4294967>  The reference bandwidth in terms of Mbits per second
</code></pre>

Use the `auto-cost reference-bandwidth` command and specify the value you want in Mbps. Let’s set it to 1000 Mbps:

<pre><code><strong>Router(config-router)#auto-cost reference-bandwidth 1000
</strong>% OSPF: Reference bandwidth is changed. 
        Please ensure reference bandwidth is consistent across all routers
</code></pre>

Cisco IOS will warn you that you should do this on all OSPF routers. Let’s verify our work:

<pre><code><strong>Router#show ip ospf | include Reference
</strong> Reference bandwidth unit is 1000 mbps
</code></pre>

Our reference bandwidth is now 1000 Mbps. Let’s see what the cost of our FastEthernet is now:

<pre><code><strong>Router#show ip ospf interface FastEthernet 0/0 | include Cost
</strong>  Process ID 1, Router ID 192.168.1.1, Network Type BROADCAST, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
</code></pre>

It now has a cost of 10 which means that a Gigabit interface would end up with a cost of 1.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of R1.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet0/0
 ip address 192.168.1.1 255.255.255.0
!
interface Serial0/0
 ip address 192.168.2.1 255.255.255.0
!
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.2.0 0.0.0.255 area 0
 auto-cost reference-bandwidth 1000
!
end
```
{% endtab %}
{% endtabs %}

That’s all there is to it. I hope this example has been useful in understanding how OSPF calculates its metric. If you have any questions, feel free to leave a comment.
