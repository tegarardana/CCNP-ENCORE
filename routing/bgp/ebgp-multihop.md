# eBGP Multihop

[eBGP (external BGP)](https://networklessons.com/cisco/ccnp-encor-350-401/how-to-configure-ebgp-external-bgp) by default requires two Cisco IOS routers to be directly connected to each other in order to establish a neighbor adjacency. This is because eBGP routers use a **TTL of one** for their BGP packets. When the BGP neighbor is more than one hop away, the TTL will decrement to 0 and it will be discarded.

When these two routers are not directly connected then we can still make it work but we’ll have to use **multihop**. This requirement does not apply to internal BGP.

Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/BGP-as1-as3-r1-r3.png" alt=""><figcaption></figcaption></figure>

Above we will try to configure eBGP between R1 and R3. Since R2 is in the middle, these routers are more than one hop away from each other. Let’s take a look at the configuration:

<pre><code><strong>R1(config)#ip route 192.168.23.3 255.255.255.255 192.168.12.2
</strong></code></pre>

<pre><code><strong>R3(config)#ip route 192.168.12.1 255.255.255.255 192.168.23.2
</strong></code></pre>

First I will create some static routes so that R1 and R3 are able to reach each other. Now we can configure eBGP:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.23.3 remote-as 3
</strong></code></pre>

<pre><code><strong>R3(config)#router bgp 3
</strong><strong>R3(config-router)#neighbor 192.168.12.1 remote-as 1
</strong></code></pre>

Even though this configuration is correct, BGP will not even try to establish a eBGP neighbor adjacency. BGP knows that since these routers are on different subnets, they are not directly connected. We can verify this with the following command:

<pre><code><strong>R1#show ip bgp neighbors | include External
</strong>  External BGP neighbor not directly connected.
</code></pre>

<pre><code><strong>R3#show ip bgp neighbors | include External
</strong>  External BGP neighbor not directly connected.
</code></pre>

Just for fun, let’s disable this check so that R1 and R3 try to become eBGP neighbors. We can do that like this:

<pre><code><strong>R1(config-router)#neighbor 192.168.23.3 disable-connected-check
</strong></code></pre>

<pre><code><strong>R3(config-router)#neighbor 192.168.12.1 disable-connected-check
</strong></code></pre>

Our routers will now try to become eBGP neighbors even though they are not directly connected. Here’s what happens now:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-ttl-1.png" alt=""><figcaption></figcaption></figure>

The wireshark capture above shows us that R1 is trying to connect to R3. As you can see the TTL is 1. Once R2 receives this packet it will decrement the TTL by 1 and drop it:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-ttl-icmp-ttl-exceeded.png" alt=""><figcaption></figcaption></figure>

Above you can see that R2 is dropping this packet since the TTL is exceeded. It will send an ICMP time-to-live exceeded message to R1. Our BGP routers will show a message like this:

```
R1#
BGP: 192.168.23.3 open failed: Connection timed out; remote host not responding, open active delayed 27593ms (35000ms max, 28% jitter)
```

This is R1 telling us that it couldn’t connect to R3. To fix this issue, we’ll tell eBGP to increase the TTL. First let’s enable the directly connected check again:

<pre><code><strong>R1(config-router)#no neighbor 192.168.23.3 disable-connected-check
</strong></code></pre>

<pre><code><strong>R3(config-router)#no neighbor 192.168.12.1 disable-connected-check
</strong></code></pre>

And now we will increase the TTL:

<pre><code><strong>R1(config-router)#neighbor 192.168.23.3 ebgp-multihop 2
</strong></code></pre>

<pre><code><strong>R3(config-router)#neighbor 192.168.12.1 ebgp-multihop 2
</strong></code></pre>

Use the **ebgp-multihop** command to increase the TTL. Using a value of 2 is enough in our example. R2 will receive a packet with a TTL of 2, decrements it by 1 and forwards it to R3. We can verify this change by looking at the show ip bgp neighbors command:

<pre><code>R1 &#x26; R3
<strong>#show ip bgp neighbors | include External
</strong>  External BGP neighbor may be up to 2 hops away.
</code></pre>

R1 and R3 both agree that the BGP neighbor could be 2 hops away. Here’s what the BGP packet looks like in wireshark:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-ttl-2.png" alt=""><figcaption></figcaption></figure>

This capture shows us the TTL of 2. After a few seconds, our routers will become eBGP neighbors:

```
R1#
%BGP-5-ADJCHANGE: neighbor 192.168.23.3 Up
```

```
R3#
%BGP-5-ADJCHANGE: neighbor 192.168.12.1 Up
```

That’s it, problem solved!

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
ip route 192.168.23.3 255.255.255.255 192.168.12.2
!
router bgp 1
 neighbor 192.168.23.3 remote-as 3
 neighbor 192.168.23.3 ebgp-multihop 2
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
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
interface fastEthernet0/0
 ip address 192.168.23.3 255.255.255.0
!
ip route 192.168.12.1 255.255.255.255 192.168.23.2
!
router bgp 3
 neighbor 192.168.12.1 remote-as 1
 neighbor 192.168.12.1 ebgp-multihop 2
!
end
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Even though R1 and R3 are now neighbors, having a non-BGP in router in between R1 and R3 is a bad idea. R1 and R3 might exchange prefixes through BGP but once packets reach R2, it will have no clue where to forward these packets to…
{% endhint %}

Now you understand how eBGP multihop works, let’s take a look at a more useful scenario:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-r1-r2-dual-link-multihop.png" alt=""><figcaption></figcaption></figure>

Above we have two routers…R1 and R2. They are directly connected but we have two links in between them and we would like to use these for load balancing. Instead of using the IP addresses on these FastEthernet interfaces for the eBGP neighbor adjacency we will use the IP addresses on the loopback interfaces for this. Let’s take a look at the configuration:

<pre><code><strong>R1(config)#ip route 2.2.2.0 255.255.255.0 192.168.12.2
</strong><strong>R1(config)#ip route 2.2.2.0 255.255.255.0 192.168.21.2
</strong></code></pre>

<pre><code><strong>R2(config)#ip route 1.1.1.0 255.255.255.0 192.168.12.1
</strong><strong>R2(config)#ip route 1.1.1.0 255.255.255.0 192.168.21.1
</strong></code></pre>

On each router we will configure two static routes, this allows us to use load balancing to reach the loopback interfaces. Now we can configure eBGP:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 2.2.2.2 remote-as 2
</strong><strong>R1(config-router)#neighbor 2.2.2.2 update-source loopback 0
</strong><strong>R1(config-router)#neighbor 2.2.2.2 ebgp-multihop 2
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 1.1.1.1 remote-as 1
</strong><strong>R2(config-router)#neighbor 1.1.1.1 update-source loopback 0
</strong><strong>R2(config-router)#neighbor 1.1.1.1 ebgp-multihop 2
</strong></code></pre>

Besides configuring the TTL to 2 with the ebgp-multihop command we also have to use the **update-source** command to tell the routers to use the IP address on their loopback interface as the source IP address for the eBGP neighbor adjacency. After a few seconds, these routers will become neighbors:

```
R1#
%BGP-5-ADJCHANGE: neighbor 2.2.2.2 Up
```

```
R2#
%BGP-5-ADJCHANGE: neighbor 1.1.1.1 Up
```

Thanks to our static routes, we will use load balancing between the two routers:

<pre><code><strong>R1#show ip route static
</strong>     2.0.0.0/24 is subnetted, 1 subnets
S       2.2.2.0 [1/0] via 192.168.21.2
                [1/0] via 192.168.12.2
</code></pre>

<pre><code><strong>R2#show ip route static
</strong>     1.0.0.0/24 is subnetted, 1 subnets
S       1.1.1.0 [1/0] via 192.168.21.1
                [1/0] via 192.168.12.1
</code></pre>

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
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
interface fastEthernet0/1
 ip address 192.168.21.1 255.255.255.0
!
ip route 2.2.2.0 255.255.255.0 192.168.12.2
ip route 2.2.2.0 255.255.255.0 192.168.21.2
!
router bgp 1
 neighbor 2.2.2.2 remote-as 2
 neighbor 2.2.2.2 update-source loopback 0
 neighbor 2.2.2.2 ebgp-multihop 2
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
interface fastEthernet0/1
 ip address 192.168.21.2 255.255.255.0
!
ip route 1.1.1.0 255.255.255.0 192.168.12.1
ip route 1.1.1.0 255.255.255.0 192.168.21.1
!
router bgp 2
 neighbor 1.1.1.1 remote-as 1
 neighbor 1.1.1.1 update-source loopback 0
 neighbor 1.1.1.1 ebgp-multihop 2
!
end
```
{% endtab %}
{% endtabs %}

That’s all there is to it. If you have any questions, feel free to leave a comment!
