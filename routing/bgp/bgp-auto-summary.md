# BGP Auto-Summary

In a previous lesson I explained how the [BGP network command](https://networklessons.com/cisco/ccnp-encor-350-401/how-to-advertise-networks-in-bgp) works. When we enable auto-summary for BGP, the way the network command works changes slightly.

Normally when you advertise a network in BGP you have to type in the **exact network and subnet mask** that you want to advertise or it won’t be placed in the BGP table.

With auto-summary enabled, you can advertise a classful network and you don’t have to add the **mask** **parameter**. BGP will automatically advertise the classful network if you have the classful network or a subnet of this network in your routing table. Let me give you an example to explain what I’m talking about. I’ll use these two routers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-r1-r2-as1-as2-topology.png" alt=""><figcaption></figcaption></figure>

These routers are configured for eBGP, there’s a loopback interface on R1 with network 1.1.1.1 /32. Here’s the configuration:

<pre><code><strong>R1#show run | section bgp
</strong>router bgp 1
 bgp log-neighbor-changes
 neighbor 192.168.12.2 remote-as 2
</code></pre>

<pre><code><strong>R2#show run | section bgp
</strong>router bgp 2
 bgp log-neighbor-changes
 neighbor 192.168.12.1 remote-as 1
</code></pre>

The configuration is straight-forward, we only configured eBGP, no networks have been advertised and auto-summary is disabled. Let’s see if we can advertise classful network 1.0.0.0/8:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 1.0.0.0
</strong></code></pre>

Note that I didn’t specify a subnet mask with the mask parameter. Take a look at the BGP table now:

<pre><code><strong>R1#show ip bgp 1.0.0.0
</strong>% Network not in table
</code></pre>

As expected there is nothing in the BGP table since we require the exact network and subnet mask. Let’s enable auto-summary now so you can see the difference:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#auto-summary
</strong></code></pre>

After enabling auto-summary things will change. Take a look at the BGP table of R1:

<pre><code><strong>R1#show ip bgp 1.0.0.0
</strong>BGP routing table entry for 1.0.0.0/8, version 2
Paths: (1 available, best #1, table default)
  Advertised to update-groups:
     1
  Local
    0.0.0.0 from 0.0.0.0 (192.168.12.1)
      Origin IGP, metric 0, localpref 100, weight 32768, valid, sourced, local, best
</code></pre>

R1 now has an entry for classful network 1.0.0.0/8. It was able to install this in its BGP table because auto-summary is enabled and we have 1.1.1.1/32 in our routing table. This network will also be advertised to R2:

<pre><code><strong>R2#show ip bgp
</strong>BGP table version is 2, local router ID is 192.168.12.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.0.0.0          192.168.12.1             0             0 1 i
</code></pre>

That’s all there is to it.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface Loopback 0
 ip address 1.1.1.1 255.255.255.255
!
interface fastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 network 1.0.0.0
 auto-summary
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface fastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
router bgp 2
 neighbor 192.168.12.1 remote-as 1
!
end
```
{% endtab %}
{% endtabs %}

I hope this example has been useful, if you have any questions feel free to leave a comment!
