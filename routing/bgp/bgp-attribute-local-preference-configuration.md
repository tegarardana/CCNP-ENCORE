# BGP Attribute Local Preference Configuration

BGP attribute local preference is the second BGP attribute and it can be used to choose the exit path for an autonomous system. Here are the details:

* Local preference is the second BGP attribute.
* You can use local preference to choose the outbound external BGP path.
* Local preference is sent to all internal BGP routers in your autonomous system.
* Not exchanged between external BGP routers.
* Local preference is a well-known and discretionary BGP attribute.
* The default value is 100.
* The path with the highest local preference is preferred

Let me show you an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/bgp-local-preference-example-1.png" alt=""><figcaption></figcaption></figure>

You can use local preference to configure your autonomous system to select a certain exit point. Instead of configuring weight on each router, you can use local preference because it is exchanged on all internal BGP routers. By increasing the local preference to 800, we can make AS 1 send all traffic toward AS 2.

A well-known discretionary BGP attribute must be recognized by all BGP routers per RFC, but its presence in a BGP update is optional.

## Configuration

Now let me show you how to configure local preference. Here is the topology that we will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/bgp-local-preference-lab.png" alt=""><figcaption></figcaption></figure>

In the picture above, we have two autonomous systems. R1 will advertise network 1.1.1.0/24 towards AS 2, and R4 will have to choose when it wants to reach this network. It can go through router R2 or R3. We’ll see how local preference influence this. Here’s the default BGP configuration of R1:

<pre><code><strong>R2(config)#interface loopback 0
</strong><strong>R2(config-if)#ip address 1.1.1.1 255.255.255.0
</strong>
<strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong><strong>R1(config-router)#neighbor 192.168.13.3 remote-as 2
</strong><strong>R1(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

Let’s configure AS 2 with OSPF:

<pre><code><strong>R2(config)#interface loopback 0
</strong><strong>R2(config-if)#ip address 2.2.2.2 255.255.255.0
</strong>
<strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 192.168.24.0 0.0.0.255 area 0
</strong><strong>R2(config-router)#network 2.2.2.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R3(config)#interface loopback 0
</strong><strong>R3(config-if)#ip address 3.3.3.3 255.255.255.0
</strong>
<strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#network 192.168.34.0 0.0.0.255 area 0
</strong><strong>R3(config-router)#network 3.3.3.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R4(config)#interface loopback 0
</strong><strong>R4(config-if)#ip address 4.4.4.4 255.255.255.0
</strong>
<strong>R4(config)#router ospf 1
</strong><strong>R4(config-router)#network 192.168.24.0 0.0.0.255 area 0
</strong><strong>R4(config-router)#network 192.168.34.0 0.0.0.255 area 0
</strong><strong>R4(config-router)#network 4.4.4.0 0.0.0.255 area 0
</strong></code></pre>

Now we can configure IBGP within AS 2:

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong><strong>R2(config-router)#neighbor 3.3.3.3 remote-as 2
</strong><strong>R2(config-router)#neighbor 3.3.3.3 update-source loopback0
</strong><strong>R2(config-router)#neighbor 4.4.4.4 remote-as 2
</strong><strong>R2(config-router)#neighbor 4.4.4.4 update-source loopback0
</strong><strong>R2(config-router)#neighbor 4.4.4.4 next-hop-self
</strong></code></pre>

<pre><code><strong>R3(config)#router bgp 2
</strong><strong>R3(config-router)#neighbor 192.168.13.1 remote-as 1
</strong><strong>R3(config-router)#neighbor 2.2.2.2 remote-as 2
</strong><strong>R3(config-router)#neighbor 2.2.2.2 update-source loopback0
</strong><strong>R3(config-router)#neighbor 4.4.4.4 remote-as 2
</strong><strong>R3(config-router)#neighbor 4.4.4.4 update-source loopback0
</strong><strong>R3(config-router)#neighbor 4.4.4.4 next-hop-self
</strong></code></pre>

<pre><code><strong>R4(config)#router bgp 2
</strong><strong>R4(config-router)#neighbor 2.2.2.2 remote-as 2
</strong><strong>R4(config-router)#neighbor 2.2.2.2 update-source loopback 0
</strong><strong>R4(config-router)#neighbor 3.3.3.3 remote-as 2
</strong><strong>R4(config-router)#neighbor 3.3.3.3 update-source loopback 0
</strong></code></pre>

And above, you can see the BGP configurations.

Now let’s find out what path R4 will use to reach network 1.1.1.0/24:

<pre><code><strong>R4#show ip bgp 
</strong>BGP table version is 2, local router ID is 4.4.4.4
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
* i1.1.1.0/24       3.3.3.3                  0    100      0 1 i
*>i                 2.2.2.2                  0    100      0 1 i
</code></pre>

All attributes are the same, so it’s the router ID that makes the decision. All traffic is sent to R2 right now. Let’s play with the local preference…

<pre><code><strong>R3(config)#router bgp 2
</strong><strong>R3(config-router)#bgp default local-preference 600
</strong></code></pre>

The **default local preference is 100,** and you can change it if you like with the **`bgp default local-preference`** command. Let’s check the BGP table:

<pre><code><strong>R4#show ip bgp 
</strong>BGP table version is 3, local router ID is 4.4.4.4
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*>i1.1.1.0/24       3.3.3.3                  0    600      0 1 i
* i                 2.2.2.2                  0    100      0 1 i
</code></pre>

Now we see that R4 prefers to send traffic to network 1.1.1.0/24 towards R3 because the local preference is 600 > 100.

Of course, we can accomplish the same thing with a route-map. Here’s how:

<pre><code><strong>R3(config)#router bgp 2
</strong><strong>R3(config-router)#no bgp default local-preference 600
</strong></code></pre>

Let’s clean up first…

<pre><code><strong>R3(config)#route-map LOCALPREF permit 10
</strong><strong>R3(config-route-map)#set local-preference 700
</strong>
<strong>R3(config)#router bgp 2
</strong><strong>R3(config-router)#neighbor 192.168.13.1 route-map LOCALPREF in
</strong></code></pre>

Route-maps are a more flexible solution. If you don’t use a match statement in a route-map, everything is matched by default. You can use it to set the local preference to another value. Don’t forget to activate the route-map by binding it to a BGP neighbor. Let’s check the BGP table:

<pre><code><strong>R4#show ip bgp 
</strong>BGP table version is 5, local router ID is 4.4.4.4
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*>i1.1.1.0/24       3.3.3.3                  0    700      0 1 i
* i                 2.2.2.2                  0    100      0 1 i
</code></pre>

And as you can see above, the local preference has changed.

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
interface fastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
interface fastEthernet1/0
 ip address 192.168.13.1 255.255.255.0
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 neighbor 192.168.13.3 remote-as 2
 network 1.1.1.0 mask 255.255.255.0
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
interface fastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
interface fastEthernet2/0
 ip address 192.168.24.2 255.255.255.0
!
router ospf 1
 network 192.168.24.0 0.0.0.255 area 0
 network 2.2.2.0 0.0.0.255 area 0
!
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 neighbor 3.3.3.3 remote-as 2
 neighbor 3.3.3.3 update-source loopback0
 neighbor 4.4.4.4 remote-as 2
 neighbor 4.4.4.4 update-source loopback0
 neighbor 4.4.4.4 next-hop-self
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
interface Loopback 0
ip address 3.3.3.3 255.255.255.0
!
interface fastEthernet0/0
ip address 192.168.13.3 255.255.255.0
!
interface fastEthernet2/0
ip address 192.168.34.3 255.255.255.0
!
router ospf 1
 network 192.168.34.0 0.0.0.255 area 0
 network 3.3.3.0 0.0.0.255 area 0
!
router bgp 2
 neighbor 192.168.13.1 remote-as 1
 neighbor 2.2.2.2 remote-as 2
 neighbor 2.2.2.2 update-source loopback0
 neighbor 4.4.4.4 remote-as 2
 neighbor 4.4.4.4 update-source loopback0
 neighbor 4.4.4.4 next-hop-self
 neighbor 192.168.13.1 route-map LOCALPREF in
!
route-map LOCALPREF permit 10
 set local-preference 700
!
end
```
{% endtab %}

{% tab title="R4" %}
```
hostname R4
!
interface Loopback 0
 ip address 4.4.4.4 255.255.255.0
!
interface fastEthernet0/0
 ip address 192.168.24.4 255.255.255.0
!
interface fastEthernet1/0
 ip address 192.168.34.4 255.255.255.0
!
router ospf 1
 network 192.168.24.0 0.0.0.255 area 0
 network 192.168.34.0 0.0.0.255 area 0
 network 4.4.4.0 0.0.0.255 area 0
!
router bgp 2
 neighbor 2.2.2.2 remote-as 2
 neighbor 2.2.2.2 update-source loopback 0
 neighbor 3.3.3.3 remote-as 2
 neighbor 3.3.3.3 update-source loopback 0
!
end
```
{% endtab %}
{% endtabs %}

I hope you enjoyed reading this lesson. If you have any more questions, feel free to ask
