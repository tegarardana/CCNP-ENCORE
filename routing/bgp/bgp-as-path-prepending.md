# BGP AS-Path Prepending

The fourth BGP attribute is called AS Path:

* BGP prefers the shortest AS path to get to a destination. Less is more!
* We can manipulate this by using AS path prepending.

Let me show you an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/bgp-as-path-prepend.png" alt=""><figcaption></figcaption></figure>

In my example AS 1 wants to make sure traffic enters the autonomous system through R2. We can add our own autonomous system number multiple times, so the as-path becomes longer. Since BGP prefers a shorter AS path, we can influence our routing. This is called AS path pretending.

## Configuration

Let’s see what this looks like on Cisco routers. This is the topology that I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/bgp-as-path-prepend-lab.png" alt=""><figcaption></figcaption></figure>

Above we have 3 routers. R1 and R3 are both in AS 1 advertising the same network (1.1.1.0/24) to R2. We can use AS Path prepending to make R2 prefer a certain path.

Let’s start with a BGP configuration:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#bgp router-id 1.1.1.1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong></code></pre>

<pre><code><strong>R3(config)#router bgp 1
</strong><strong>R3(config-router)#bgp router-id 3.3.3.3
</strong><strong>R3(config-router)#neighbor 192.168.23.2 remote-as 2
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong><strong>R2(config-router)#neighbor 192.168.23.3 remote-as 1
</strong></code></pre>

This is just a basic configuration. Now we can advertise network 1.1.1.0 /24 on both R1 and R3:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

<pre><code><strong>R3(config)#router bgp 1
</strong><strong>R3(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

Let’s check the BGP table:

<pre><code><strong>R2#show ip bgp 
</strong>BGP table version is 2, local router ID is 192.168.23.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  1.1.1.0/24       192.168.23.3             0             0 1 i
*>                  192.168.12.1             0             0 1 i
</code></pre>

In the table above, you can see that it prefers 192.168.12.1 as its path. Let’s change the AS path so that we’ll use 192.168.23.3 as the preferred path. Here is an example:

<pre><code><strong>R1(config)#route-map PREPEND permit 10
</strong><strong>R1(config-route-map)#set as-path prepend 1 1 1 1 1
</strong><strong>R1(config-route-map)#exit
</strong><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 route-map PREPEND out
</strong></code></pre>

First, create a route-map and use **`set as-path prepend`** to add your own AS number multiple times.\
Don’t forget to add the route-map to your BGP neighbor configuration, and since you are sending this to your remote neighbor, it should be **outbound**! Let’s check the BGP table:

<pre><code><strong>R2#show ip bgp 
</strong>BGP table version is 2, local router ID is 192.168.23.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.23.3             0             0 1 i
*                   192.168.12.1             0             0 1 1 1 1 1 1 i
</code></pre>

Now we see that 192.168.23.3 is our next hop IP address. You can also see that the AS Path has become longer for the second entry. That’s it!

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
 bgp router-id 1.1.1.1
 neighbor 192.168.12.2 remote-as 2
 network 1.1.1.0 mask 255.255.255.0
 neighbor 192.168.12.2 route-map PREPEND out
!
route-map PREPEND permit 10
 set as-path prepend 1 1 1 1 1
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
 bgp router-id 3.3.3.3
 neighbor 192.168.23.2 remote-as 2
 network 1.1.1.0 mask 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

That’s all there is for now! If you have any questions, leave a comment…
