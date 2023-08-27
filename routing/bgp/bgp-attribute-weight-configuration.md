# BGP Attribute Weight Configuration

Weight is a Cisco proprietary BGP attribute that can be used to select a certain path. Here’s what you need to know about weight:

* Weight is the first BGP attribute in the list.
* Cisco proprietary so you won’t find it on other vendor routers.
* Weight is not exchanged between BGP routers.
* Weight is only local on the router.
* The path with the highest weight is preferred.

Let me give you an example of BGP weight:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/bgp-weight-prefer-path.png" alt=""><figcaption></figcaption></figure>

R1 in AS 1 can reach AS 3 through AS 2 or AS 4. If we want to ensure AS 2 is always used as the best path, you can change the weight. In my example, the weight for the path to AS 2 is set to 500 and higher than the weight of 400 for AS 4. Let’s see what this looks like on real Cisco routers. This is the topology that I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/bgp-weight-attribute-lab-topology.png" alt=""><figcaption></figcaption></figure>

Above, we have a simple scenario with two autonomous systems. R2 and R3 both have network 2.2.2.0/24 configured on their loopback0 interface, and I’ll advertise that in BGP.

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#bgp router-id 1.1.1.1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong><strong>R1(config-router)#neighbor 192.168.13.3 remote-as 2
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#bgp router-id 2.2.2.2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong><strong>R2(config-router)#neighbor 192.168.23.3 remote-as 2
</strong><strong>R2(config-router)#network 2.2.2.0 mask 255.255.255.0
</strong></code></pre>

<pre><code><strong>R3(config)#router bgp 2
</strong><strong>R3(config-router)#bgp router-id 3.3.3.3
</strong><strong>R3(config-router)#neighbor 192.168.13.1 remote-as 1
</strong><strong>R3(config-router)#neighbor 192.168.23.2 remote-as 2
</strong><strong>R3(config-router)#network 2.2.2.0 mask 255.255.255.0
</strong></code></pre>

Above you’ll find the configuration for BGP. I configured the manual router ID for a reason. R2 and R3 have the same IP address on the loopback interface, which means they would get the same router ID, and they would be unable to form a BGP neighbor adjacency. Let’s take a detailed look at R1:

<pre><code><strong>R1#show ip bgp 
</strong>BGP table version is 2, local router ID is 192.168.13.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 2.2.2.0/24       192.168.12.2             0             0 2 i
*                   192.168.13.3             0             0 2 i
</code></pre>

Router R1 decided to use 192.168.12.2 as the next hop. All the BGP attributes are the same, so it came down to the router ID to select a winner.

{% hint style="info" %}
The default weight for a prefix that the router originates is 32768. You can verify this by taking a look at prefix 2.2.2.0/24 in the BGP table on R2 or R3.
{% endhint %}

Now let’s change this behavior using the weight attribute…

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.13.3 weight 500
</strong></code></pre>

You can configure weight **per neighbor** using the **weight** command. All prefixes from this neighbor will have a weight of 500.

<pre><code><strong>R1#clear ip bgp *
</strong></code></pre>

Sometimes BGP behaves like an oil tanker, so to speed things up in your lab, reset it.

<pre><code><strong>R1#show ip bgp   
</strong>BGP table version is 2, local router ID is 192.168.13.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  2.2.2.0/24       192.168.12.2             0             0 2 i
*>                  192.168.13.3             0           500 2 i
</code></pre>

You can see that 192.168.13.3 has been selected as the next hop because the weight is now 500.

What if I want to set the weight to 500 for just a couple of prefixes from AS 2?

<pre><code><strong>R2(config)#interface loopback 1
</strong><strong>R2(config-if)#ip address 22.22.22.22 255.255.255.0
</strong>
<strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#network 22.22.22.0 mask 255.255.255.0
</strong></code></pre>

<pre><code><strong>R3(config)#interface loopback 1
</strong><strong>R3(config-if)#ip address 22.22.22.22 255.255.255.0
</strong>
<strong>R3(config)#router bgp 2
</strong><strong>R3(config-router)#network 22.22.22.0 mask 255.255.255.0
</strong></code></pre>

I’ll create a new loopback interface on routers R2 and R3, and I’ll advertise network 22.22.22.0/24 in BGP. Here’s what router R1 now looks like:

<pre><code><strong>R1#show ip bgp 
</strong>BGP table version is 5, local router ID is 192.168.13.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  2.2.2.0/24       192.168.12.2             0             0 2 i
*>                  192.168.13.3             0           500 2 i
*> 22.22.22.0/24    192.168.13.3             0           500 2 i
*                   192.168.12.2             0             0 2 i
</code></pre>

As you can see above, router R1 will use 192.168.13.3 as the next hop for both prefixes. What if I want to change the weight for just one prefix? Route-maps to the rescue!

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#no neighbor 192.168.13.3 weight 500
</strong></code></pre>

First, we’ll get rid of the command above.

<pre><code><strong>R1(config)#route-map SETWEIGHT permit 10
</strong><strong>R1(config-route-map)#match ip address 1
</strong><strong>R1(config-route-map)#set weight 400
</strong><strong>R1(config-route-map)#exit
</strong><strong>R1(config)#route-map SETWEIGHT permit 20
</strong><strong>R1(config-route-map)#set weight 0
</strong><strong>R1(config-route-map)#exit
</strong><strong>R1(config)#access-list 1 permit 22.22.22.0 0.0.0.255
</strong></code></pre>

Here’s the route-map that I will use. If the prefixes match access-list 1, we will set the weight to 400.

<pre><code><strong>R1(config-router)#neighbor 192.168.13.3 route-map SETWEIGHT in
</strong></code></pre>

To complete the configuration, we have to apply it to our neighbor in AS 2. Using a route-map gives you a lot of control!

<pre><code><strong>R1#clear ip bgp *
</strong></code></pre>

This will speed things up…

<pre><code><strong>R1#show ip bgp 
</strong>BGP table version is 3, local router ID is 192.168.13.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  2.2.2.0/24       192.168.13.3             0             0 2 i
*>                  192.168.12.2             0             0 2 i
*> 22.22.22.0/24    192.168.13.3             0           400 2 i
*                   192.168.12.2             0             0 2 i
</code></pre>

See how the weight changed for network 22.22.22.0/24 ? Use route-maps to influence the BGP attributes per neighbor/prefix.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface fastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
interface fastEthernet1/0
 ip address 192.168.13.1 255.255.255.0
!
router bgp 1
 bgp router-id 1.1.1.1
 neighbor 192.168.12.2 remote-as 2
 neighbor 192.168.13.3 remote-as 2
 neighbor 192.168.13.3 route-map SETWEIGHT in
!
route-map SETWEIGHT permit 10
 match ip address 1
 set weight 400
!
route-map SETWEIGHT permit 20
 set weight 0
!
access-list 1 permit 22.22.22.0 0.0.0.255
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface Loopback 0
 ip address 2.2.2.2 255.255.255.0
!
interface Loopback 1
 ip address 22.22.22.22 255.255.255.0
!
interface fastEthernet1/0
 ip address 192.168.12.2 255.255.255.0
!
interface fastEthernet2/0
 ip address 192.168.23.2 255.255.255.0
!
router bgp 2
 bgp router-id 2.2.2.2
 neighbor 192.168.12.1 remote-as 1
 neighbor 192.168.23.3 remote-as 2
 network 2.2.2.0 mask 255.255.255.0
 network 22.22.22.0 mask 255.255.255.0
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
interface Loopback 0
 ip address 2.2.2.2 255.255.255.0
!
interface loopback 1
 ip address 22.22.22.22 255.255.255.0
!
interface fastEthernet0/0
 ip address 192.168.13.3 255.255.255.0
!
interface fastEthernet1/0
 ip address 192.168.23.3 255.255.255.0
!
router bgp 2
 bgp router-id 3.3.3.3
 neighbor 192.168.13.1 remote-as 1
 neighbor 192.168.23.2 remote-as 2
 network 2.2.2.0 mask 255.255.255.0
 network 22.22.22.0 mask 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

And that’s the end of it. I hope this has been helpful for you in understanding the BGP weight attribute! If you have any questions, leave a comment.
