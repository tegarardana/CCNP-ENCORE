# Floating Static Route

Static routes have a very low administrative distance of 1, this means that your router will prefer a static route over any routes that were learned through a dynamic routing protocol. If we want to use a static route as a backup route, we’ll have to change its administrative distance. This is called a **floating static route**.

In this lesson, I’ll show you how to do this. We will use the following topology for this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/01/r1-r2-r3-triangle.png" alt=""><figcaption></figcaption></figure>

R1 can use R2 or R3 to get to the 192.168.23.0/24 network.

## Configuration

Let’s configure R1 and R2 to use RIP so that it can reach this network:

<pre><code><strong>R1(config)#router rip
</strong><strong>R1(config-router)#version 2
</strong><strong>R1(config-router)#no auto-summary 
</strong><strong>R1(config-router)#network 192.168.12.0
</strong></code></pre>

<pre><code><strong>R2(config-router)#version 2
</strong><strong>R2(config-router)#no auto-summary 
</strong><strong>R2(config-router)#network 192.168.12.0
</strong><strong>R2(config-router)#network 192.168.23.0
</strong></code></pre>

R1 should now be able to reach this network through R2:

<pre><code><strong>R1#show ip route | begin 192.168.23.0
</strong>R     192.168.23.0/24 [120/1] via 192.168.12.2, 00:00:22, GigabitEthernet0/1
</code></pre>

There we go, R1 has a route. What if we want to use R3 as a backup? R3 is not running any routing protocols, so we have to use a static route. This is a possibility when the router is outside of your control.

Let’s create a static route for 192.168.23.0/24 through R3:

<pre><code><strong>R1(config)#ip route 192.168.23.0 255.255.255.0 192.168.13.3
</strong></code></pre>

This static route works but there’s one catch, check the routing table:

<pre><code><strong>R1#show ip route | begin 192.168.23.0
</strong>
S     192.168.23.0/24 [1/0] via 192.168.13.3
</code></pre>

Because of the lower administrative distance (1 for a static route), our RIP route is now gone…let’s remove this static route:

<pre><code><strong>R1(config)#no ip route 192.168.23.0 255.255.255.0 192.168.13.3
</strong></code></pre>

The trick is that you need to configure an AD when you configure your static route. Here’s how:

<pre><code><strong>R1(config)#ip route 192.168.23.0 255.255.255.0 192.168.13.3 ?
</strong>  &#x3C;1-255>    Distance metric for this route
  multicast  multicast route
  name       Specify name of the next hop
  permanent  permanent route
  tag        Set tag for this route
  track      Install route depending on tracked item
  &#x3C;cr>
</code></pre>

The administrative distance of RIP is 120 so if we pick a higher number, our static route will be used as a backup. Let’s try 121:

<pre><code><strong>R1(config)#ip route 192.168.23.0 255.255.255.0 192.168.13.3 121
</strong></code></pre>

When we now check our routing table:

<pre><code><strong>R1#show ip route | begin 192.168.23.0
</strong>
R     192.168.23.0/24 [120/1] via 192.168.12.2, 00:00:26, GigabitEthernet0/1
</code></pre>

We see that the RIP entry is used again. The static route is still there somewhere behind the scenes…let’s see if that is true. To test this, I will shut the interface on R2 that connects to R1:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#shutdown
</strong></code></pre>

RIP is a pretty slow routing protocol, so you have to wait awhile for the route to disappear. After awhile, you will see the changes in the routing table of R1:

<pre><code><strong>R1#show ip route | begin 192.168.23.0
</strong>S     192.168.23.0/24 [121/0] via 192.168.13.3
</code></pre>

There it is, our floating static route is now installed in the routing table. You can see it shows the AD of 121. If you would un-shut the interface of R2, it will install the RIP route again.\


{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 192.168.13.1 255.255.255.0
!
router rip
 network 192.168.12.0
 no auto-summary
!
ip route 192.168.23.0 255.255.255.0 192.168.13.3 121
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
!
router rip
 network 192.168.12.0
 network 192.168.23.0
 no auto-summary
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.13.3 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 192.168.23.3 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

You have now learned how you can configure static routes with a higher administrative distance, turning them into floating static routes. You can use these as a backup for routes that were learned through dynamic routing protocols (or other static routes with a lower administrative distance).
