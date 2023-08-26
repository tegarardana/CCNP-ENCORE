# OSPF Hello and Dead Interval

OSPF uses hello packets and two timers to check if a neighbor is still alive or not:

* Hello interval: this defines how often we send the hello packet.
* Dead interval:  this defines how long we should wait for hello packets before we declare the neighbor dead.

The hello and dead interval values can be different depending on the OSPF network type. On Ethernet interfaces you will see a 10 second hello interval and a 40 second dead interval.

Let’s take a look at an example so we can see this in action. Here’s the topology I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/ospf-r1-sw-r2.png" alt=""><figcaption></figcaption></figure>

We’ll use two routers with a switch in between.

## Configuration

Let’s enable OSPF:

<pre><code>R1 &#x26; R2#
<strong>(config)#router ospf 1
</strong><strong>(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong></code></pre>

Let’s take a look at the default hello and dead interval:

<pre><code><strong>R1#show ip ospf interface FastEthernet 0/0 | include intervals
</strong>  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
</code></pre>

The hello and dead interval can be different for each interface. Above you can see that the hello interval is 10 seconds and the dead interval is 40 seconds. Let’s try if this is true:

<pre><code><strong>R1(config)#interface FastEthernet 0/0
</strong><strong>R1(config-if)#shutdown 
</strong></code></pre>

After shutting the interface on R1 you will see the following message:

```
R1#
Aug 30 17:57:05.519: %OSPF-5-ADJCHG: Process 1, Nbr 192.168.12.2 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: Interface down or detached
```

R1 will know that R2 is unreachable since its interface went down. Now take a look at R2:

```
R2#
Aug 30 17:57:40.863: %OSPF-5-ADJCHG: Process 1, Nbr 192.168.12.1 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: Dead timer expired
```

R2 is telling us that the dead timer has expired. This took a bit longer. The interface on R1 went down at 17:57:05 and R2’s dead timer expired at 17:57:40…that’s close to 40 seconds.

Let’s activate the interface again:

<pre><code><strong>R1(config)#interface FastEthernet 0/0
</strong><strong>R1(config-if)#no shutdown
</strong></code></pre>

40 seconds is a long time…R2 will keep sending traffic to R1 while the dead interval is expiring. To speed up this process we can play with the timers. Here’s an example:

<pre><code>R1 &#x26; R2
<strong>(config)#interface FastEthernet 0/0
</strong><strong>(config-if)#ip ospf hello-interval 1 
</strong><strong>(config-if)#ip ospf dead-interval 3
</strong></code></pre>

You can use these two commands to change the hello and dead interval. We’ll send a hello packet every second and the dead interval is 3 seconds. Let’s verify this:

<pre><code><strong>R1#show ip ospf interface FastEthernet 0/0 | include intervals
</strong>  Timer intervals configured, Hello 1, Dead 3, Wait 3, Retransmit 5
</code></pre>

Reducing the dead interval from 40 to 3 seconds is a big improvement but we can do even better:

<pre><code><strong>R1(config-if)#ip ospf dead-interval ?
</strong>  &#x3C;1-65535>  Seconds
  minimal    Set to 1 second
</code></pre>

We can reduce the dead interval to 1 second. If you use the ip ospf dead-interval minimal command then you have to specify the number of hellos sent in one second:

<pre><code><strong>R1(config-if)#ip ospf dead-interval minimal hello-multiplier ?
</strong>  &#x3C;3-20>  Number of Hellos sent within 1 second
</code></pre>

Let’s change it to 3 hello packets:

<pre><code>R1 &#x26; R2
<strong>(config-if)#ip ospf dead-interval minimal hello-multiplier 3
</strong></code></pre>

We now have superfast hello packets. Take a look below:

<pre><code><strong>R1#show ip ospf interface FastEthernet 0/0 | include intervals
</strong>  Timer intervals configured, Hello 333 msec, Dead 1, Wait 1, Retransmit 5
</code></pre>

Each 333 msec we will send a hello packet, if we don’t receive any within a second then we will declare the neighbor dead.

Our routers will now react quickly when they don’t receive a hello packet in time.

{% hint style="info" %}
It’s possible to reduce failover times even more using something called [Bidirectional Forwarding Detection (BFD)](https://networklessons.com/cisco/ccnp-encor-350-401/bidirectional-forwarding-detection-bfd). This is a protocol that runs independent from routing protocols and is used to detect link failures between two endpoints. It’s used often on links that don’t offer any failure detection (like Ethernet). We will cover this in another lesson.
{% endhint %}

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet 0/0
 ip address 192.168.12.1 255.255.255.0
 ip ospf hello-interval 1
 ip ospf dead-interval 3
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface FastEthernet 0/0
 ip address 192.168.12.2 255.255.255.0
 ip ospf hello-interval 1
 ip ospf dead-interval 3
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

Hopefully these examples have been useful, if you have any questions just leave a comment!
