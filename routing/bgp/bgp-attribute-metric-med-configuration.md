# BGP Attribute Metric (MED) Configuration

MED (or metric) is the sixth BGP attribute:

* MED can be used to advertise to your neighbors how they should enter your AS.
* MED is exchanged between autonomous systems.
* The lowest MED is the preferred path.
* MED is propagated to all routers within the neighbor AS but not passed along to any other autonomous systems.

Let’s look at an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/bgp-med-topology.png" alt=""><figcaption></figcaption></figure>

MED (also called metric) is exchanged between autonomous systems, and you can use it to let the other AS know which path they should use to enter your AS. R2 is sending a MED of 200 towards AS 3. R3 is sending a MED of 300 to AS 3. AS 3 will prefer the lower metric and send all traffic for AS 1 through R2.

## Configuration

Let me show you how to configure this on a Cisco router:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/bgp-as-path-prepend-lab.png" alt=""><figcaption></figcaption></figure>

Above, we have two autonomous systems. R1 and R3 will both advertise network 1.1.1.0 /24 in BGP. We can use MED to tell AS 2 which path to use to reach this network.

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong><strong>R1(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

<pre><code><strong>R3(config)#router bgp 1
</strong><strong>R3(config-router)#neighbor 192.168.23.2 remote-as 2
</strong><strong>R3(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong><strong>R2(config-router)#neighbor 192.168.23.3 remote-as 1
</strong></code></pre>

This is the BGP configuration, nothing special so far.

<pre><code><strong>R2#show ip bgp 
</strong>BGP table version is 2, local router ID is 192.168.23.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  1.1.1.0/24       192.168.23.3             0             0 1 i
*>                  192.168.12.1             0             0 1 i
</code></pre>

You have seen the example above before. R2 prefers the path through 192.168.12.1. Note that the **default metric (MED) is 0**. Let’s play with the MED now:

<pre><code><strong>R1(config)#route-map MED permit 10
</strong><strong>R1(config-route-map)#set metric 700   
</strong><strong>R1(config-route-map)#exit
</strong>
<strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 route-map MED out
</strong></code></pre>

<pre><code><strong>R3(config)#route-map MED permit 10
</strong><strong>R3(config-route-map)#set metric 500
</strong><strong>R3(config-route-map)#exit
</strong>
<strong>R3(config)#router bgp 1
</strong><strong>R3(config-router)#neighbor 192.168.23.2 route-map MED out
</strong></code></pre>

I’ll use route maps so that R1 advertises everything with a med of 700, and R3 will advertise everything with a med of 500. Let’s check the BGP table:

<pre><code><strong>R2#show ip bgp 
</strong>BGP table version is 2, local router ID is 192.168.23.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  1.1.1.0/24       192.168.12.1           700             0 1 i
*>                  192.168.23.3           500             0 1 i
</code></pre>

R2 prefers the path through 192.168.23.3 because the med is lower. That’s all there is to it.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface Loopback 0
 ip address 1.1.1.1 255.255.255.0
!
interface fastEthernet2/0
 ip address 192.168.12.1 255.255.255.0
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 network 1.1.1.0 mask 255.255.255.0
 neighbor 192.168.12.2 route-map MED out
!
route-map MED permit 10
 set metric 700
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
interface fastEthernet1/0
 ip address 192.168.23.2 255.255.255.0
!
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 neighbor 192.168.23.3 remote-as 1
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
interface Loopback 0
 ip address 1.1.1.1 255.255.255.0
!
interface fastEthernet2/0
 ip address 192.168.23.3 255.255.255.0
!
router bgp 1
 neighbor 192.168.23.2 remote-as 2
 network 1.1.1.0 mask 255.255.255.0
 neighbor 192.168.23.2 route-map MED out
!
route-map MED permit 10
 set metric 500
!
end
```
{% endtab %}
{% endtabs %}

I hope this has helped you to understand BGP MED. If you have any questions, please leave a comment.
