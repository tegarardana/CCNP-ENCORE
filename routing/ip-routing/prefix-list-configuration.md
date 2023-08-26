# Prefix List Configuration

Prefix-lists can be used to filter prefixes and are far more powerful than simple access-lists. Let’s say I want to filter all prefixes that fall within the 10.0.0.0 range and that have a subnet mask between /24 and /28. Do you think you could do this with an access-list? It won’t be easy, right…with a prefix-list this is very easy to do!

Most CCNP students find prefix-lists difficult to understand so in this lesson I’ll show you how prefix-lists work by using them as route filters.\


I will show you different scenarios and different filters. Here is the topology that we will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/prefix-list-example-topology.png" alt=""><figcaption></figcaption></figure>

Above you see two routers called “R1” and “R2”. On R2, we have a couple of loopback interfaces with prefixes that we will advertise in EIGRP. I’m doing this, so we have several prefixes to play with. Here is the configuration:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#no auto-summary
</strong><strong>R1(config-router)#network 192.168.12.0
</strong></code></pre>

<pre><code><strong>R2(config)#router eigrp 12
</strong><strong>R2(config-router)#no auto-summary
</strong><strong>R2(config-router)#network 192.168.12.0
</strong><strong>R2(config-router)#network 172.16.0.0 0.0.3.255
</strong></code></pre>

EIGRP is configured, so all networks are advertised.

<pre><code><strong>R1#show ip route eigrp 
</strong>     172.16.0.0/24 is subnetted, 4 subnets
D       172.16.0.0 [90/156160] via 192.168.12.2, 00:01:07, FastEthernet0/0
D       172.16.1.0 [90/156160] via 192.168.12.2, 00:01:07, FastEthernet0/0
D       172.16.2.0 [90/156160] via 192.168.12.2, 00:01:07, FastEthernet0/0
D       172.16.3.0 [90/156160] via 192.168.12.2, 00:01:07, FastEthernet0/0
</code></pre>

If we look at the routing table of R1 we can see all those networks on the loopback interfaces as they should be. Now we’ll see if we can do some filtering. Let’s start with a simple prefix-list that filters 172.16.1.0 /24 but permits everything else:

<pre><code><strong>R1(config)#ip prefix-list FILTERTHIS seq 5 deny 172.16.1.0/24
</strong><strong>R1(config)#ip prefix-list FILTERTHIS seq 10 permit 0.0.0.0/0 le 32
</strong></code></pre>

By using the **`ip prefix-list`** command, you can create prefix lists. As you can see it looks a bit similar to my access-list but instead of typing wildcards we just specify the number of bits. The first line denies 172.16.1.0/24 and the second line permits 0.0.0.0/0 (all networks) if they have a subnet mask of /32 or smaller…in other words “everything”. This line is the equivalent of `permit ip any any`.

Let’s enable it on R1 to see what the result is:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#distribute-list prefix FILTERTHIS in
</strong></code></pre>

And we’ll enable the new prefix-list.

<pre><code><strong>R1#show ip route eigrp 
</strong>     172.16.0.0/24 is subnetted, 3 subnets
D       172.16.0.0 [90/156160] via 192.168.12.2, 00:01:54, FastEthernet0/0
D       172.16.2.0 [90/156160] via 192.168.12.2, 00:01:54, FastEthernet0/0
D       172.16.3.0 [90/156160] via 192.168.12.2, 00:01:54, FastEthernet0/0
</code></pre>

As you can see, 172.16.1.0/24 has been filtered, and all the other networks are permitted.

The true power of the prefix list is in the ge (Greater than or Equal to) and le (less than or equal to) operators. Let’s look at some examples:

<pre><code><strong>R1(config)#ip prefix-list RENETEST permit 10.0.0.0/8 le 19
</strong></code></pre>

In this example, I’m using the le operator. This prefix-list statement says that all networks that fall within the 10.0.0.0/8 range AND that have a subnet mask of /19 or less are permitted.

If I have a network with 10.0.0.0 /21, it will be denied by this prefix list. It falls within the 10.0.0.0 /8 range, but it has a subnet mask of /21. I’m using the le operator, which says that the subnet mask should be /19 or smaller.

Let’s say I have another network with 10.0.0.0 /17 then it will be permitted by this prefix-list. It falls within the 10.0.0.0/8 range and has a subnet mask that is smaller than /19.

Are you following me here? Let me give you an example on our routers:

<pre><code><strong>R2(config)#interface loopback 10 
</strong><strong>R2(config-if)#ip address 10.1.1.1 255.255.0.0
</strong><strong>R2(config-if)#interface loopback 11
</strong><strong>R2(config-if)#ip address 10.2.2.2 255.255.128.0
</strong><strong>R2(config-if)#interface loopback 12
</strong><strong>R2(config-if)#ip address 10.3.3.3 255.255.192.0
</strong><strong>R2(config-if)#interface loopback 13
</strong><strong>R2(config-if)#ip address 10.4.4.4 255.255.224.0
</strong><strong>R2(config-if)#interface loopback 14
</strong><strong>R2(config-if)#ip address 10.5.5.5 255.255.240.0
</strong><strong>R2(config-if)#interface loopback 15
</strong><strong>R2(config-if)#ip address 10.6.6.6 255.255.248.0
</strong></code></pre>

First, we’ll add a couple of loopback interfaces on R2. If you look closely, you can see I’m using different subnet masks.

<pre><code><strong>R2(config)#router eigrp 12
</strong><strong>R2(config-router)#network 10.0.0.0
</strong></code></pre>

And I’ll advertise them in EIGRP.

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#no distribute-list prefix FILTERTHIS in
</strong></code></pre>

Let’s get rid of the prefix-list on R1…

<pre><code><strong>R1#show ip route eigrp 
</strong>     172.16.0.0/24 is subnetted, 4 subnets
D       172.16.0.0 [90/156160] via 192.168.12.2, 00:06:11, FastEthernet0/0
D       172.16.1.0 [90/156160] via 192.168.12.2, 00:00:35, FastEthernet0/0
D       172.16.2.0 [90/156160] via 192.168.12.2, 00:06:11, FastEthernet0/0
D       172.16.3.0 [90/156160] via 192.168.12.2, 00:06:11, FastEthernet0/0
     10.0.0.0/8 is variably subnetted, 6 subnets, 6 masks
D       10.2.0.0/17 [90/156160] via 192.168.12.2, 00:02:22, FastEthernet0/0
D       10.3.0.0/18 [90/156160] via 192.168.12.2, 01:14:57, FastEthernet0/0
D       10.1.0.0/16 [90/156160] via 192.168.12.2, 00:06:11, FastEthernet0/0
D       10.6.0.0/21 [90/156160] via 192.168.12.2, 01:02:35, FastEthernet0/0
D       10.4.0.0/19 [90/156160] via 192.168.12.2, 01:14:46, FastEthernet0/0
D       10.5.0.0/20 [90/156160] via 192.168.12.2, 01:02:35, FastEthernet0/0
</code></pre>

Now we see all the networks that fall within the 172.16.0.0/16 and 10.0.0.0/8 range. Time to enable that prefix-list I just created:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#distribute-list prefix RENETEST in
</strong></code></pre>

This is how we activate it, and this is what we end up with:

<pre><code><strong>R1#show ip route eigrp 
</strong>     10.0.0.0/8 is variably subnetted, 4 subnets, 4 masks
D       10.2.0.0/17 [90/156160] via 192.168.12.2, 00:03:27, FastEthernet0/0
D       10.3.0.0/18 [90/156160] via 192.168.12.2, 01:16:03, FastEthernet0/0
D       10.1.0.0/16 [90/156160] via 192.168.12.2, 00:07:16, FastEthernet0/0
D       10.4.0.0/19 [90/156160] via 192.168.12.2, 01:15:51, FastEthernet0/0
</code></pre>

Only four entries remain…why?

<pre><code><strong>R1#show ip prefix-list RENETEST
</strong>ip prefix-list RENETEST: 1 entries
   seq 5 permit 10.0.0.0/8 le 19
</code></pre>

Here’s the prefix-list again. Let me explain what happened:

* Everything in the 172.16.0.0/16 range is filtered because it’s not permitted in our prefix-list.
* 10.2.0.0/17 is permitted because it’s in the 10.0.0.0/8 range and has a /17 subnet mask.
* 10.3.0.0/18 is permitted because it’s in the 10.0.0.0/8 range and has a /18 subnet mask.
* 10.1.0.0/16 is permitted because it’s in the 10.0.0.0/8 range and has a /16 subnet mask.
* 10.4.0.0/16 is permitted because it’s in the 10.0.0.0/8 range and has a /19 subnet mask.
* 10.5.0.0/20 is filtered, it’s in the 10.0.0.0/8 range but has a /20 subnet mask.
* 10.6.0.0/21 is filtered, it’s in the 10.0.0.0/8 range but has a /21 subnet mask.

Does this make sense? Let’s walk through a couple more examples together!

<pre><code><strong>R1(config)#ip prefix-list RENETEST2 permit 10.0.0.0/8 ge 20
</strong></code></pre>

This time I’m using the ge operator. Ge 20 means that the network needs to have a subnet mask of /20 or larger to be permitted. 10.0.0.0 /8 is the range we are going to check.

A network with 10.55.55.0 /25 will be permitted because it falls within the 10.0.0.0 /8 range and has a subnet mask of /25, which is larger than /20.

What about 10.60.0.0 /19? It falls within the 10.0.0.0 /8 range but it is not permitted because it has a subnet mask of /19…our ge operator says it should be /20 or larger.

Hmm, interesting…what about 192.168.12.0 /25? The subnet mask of /25 matches our ge operator, but it doesn’t fall within the 10.0.0.0 /8 range, so it’s not permitted. Let’s see what happens if I activate this prefix-list on R1:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#no distribute-list prefix RENETEST in
</strong><strong>R1(config-router)#distribute-list prefix RENETEST2 in
</strong></code></pre>

First, disable the old prefix-list and secondly, enable the new one.

<pre><code><strong>R1#show ip route eigrp 
</strong>     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
D       10.6.0.0/21 [90/156160] via 192.168.12.2, 00:01:03, FastEthernet0/0
D       10.5.0.0/20 [90/156160] via 192.168.12.2, 00:01:03, FastEthernet0/0
</code></pre>

Only two entries remain…why?

* Everything in the 172.16.0.0/16 range is filtered because it’s not permitted in our prefix-list.
* All networks in the 10.0.0.0/8 range with a subnet mask that is smaller than 20 are filtered.
* All networks in the 10.0.0.0/8 range with a subnet mask that is 20 or larger are permitted, which means only 10.6.0.0/21 and 10.5.0.0/20.

<pre><code><strong>R1(config)#ip prefix-list RENETEST3 permit 10.0.0.0/8 ge 16 le 18
</strong></code></pre>

We can also combine the ge and le operators. Look at my prefix-list above. It’s permitting all networks that fall within the 10.0.0.0 /8 range and that have a subnet mask of /16, /18, and everything in between.

10.22.0.0 /18 will be permitted because it falls within the 10.0.0.0 /8 range and has a subnet mask of /18.

10.55.0.0 / 26 will be denied. It falls within the 10.0.0.0 /8 range, but the subnet mask is /26, which doesn’t match my ge or le operators.

10.4.4.0 /14 will be denied. It falls within the 10.0.0.0 /8 range, but the subnet mask is /14, which doesn’t match my ge or le operators.

192.168.12.0 /18 will be denied. It matches my ge and le operators, but it doesn’t fall within the 10.0.0.0 /8 range.

Let’s activate it on R1 and see what the result is:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#no distribute-list prefix RENETEST2 in                
</strong><strong>R1(config-router)#distribute-list prefix RENETEST3 in
</strong></code></pre>

First, we’ll remove the old prefix-list and activate the new one…

<pre><code><strong>R1#show ip route eigrp 
</strong>     10.0.0.0/8 is variably subnetted, 3 subnets, 3 masks
D       10.2.0.0/17 [90/156160] via 192.168.12.2, 00:00:36, FastEthernet0/0
D       10.3.0.0/18 [90/156160] via 192.168.12.2, 00:00:36, FastEthernet0/0
D       10.1.0.0/16 [90/156160] via 192.168.12.2, 00:00:36, FastEthernet0/0
</code></pre>

And here’s the result. What happened?

* Everything in the 172.16.0.0/16 range is filtered because it’s not permitted in our prefix-list.
* Only networks in the 10.0.0.0/8 range with a subnet mask of /16, /17, or /18 are permitted. Everything else is filtered.

Do you see how powerful these prefix-lists are? With a single line, I can create very flexible permit or deny statements! Let me show you a couple more examples of prefix-lists:

<pre><code><strong>R1(config)#ip prefix-list CLASSB permit 128.0.0.0/2 ge 17
</strong></code></pre>

This one is interesting…let’s break it down in pieces. It’s permitting 128.0.0.0 /2, and the ge operator says the subnet mask should be /17 or larger. 128.0.0.0 is the start of the class B range, and the /2 says that we have to check the first two bits. 128.0.0.0 /2 covers the entire class B network range. This prefix-list will permit any subnet in the class B network range that has a subnet mask of /17 or larger.

<pre><code><strong>R1(config)#ip prefix-list ALL permit 0.0.0.0/0 le 32
</strong></code></pre>

I showed you this one before…this one says permit 0.0.0.0 /0, which covers the entire network range. We have a le 32 operator that says the subnet mask should be /32 or smaller. What does this mean? It means its matches ALL networks!

<pre><code><strong>R1(config)#ip prefix-list DEFAULTROUTE permit 0.0.0.0/0
</strong></code></pre>

We don’t have any ge or le operators, and this prefix-list shows 0.0.0.0 /0. It’s only permitting the default route…

<pre><code><strong>R1(config)#ip prefix-list CLASSA permit 0.0.0.0/1 le 27
</strong></code></pre>

Last one…promise! The network range to check is 0.0.0.0, and we have /1, which means we are only checking the first bit. This effectively matches the whole class A range.\
We have a le operator with 27, which tells us the subnet mask should be /27 or smaller. This prefix-list matches all subnets within the class A range with a subnet mask of /27 or smaller.\


{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
router eigrp 12
 distribute-list prefix RENETEST3 in 
 network 192.168.12.0
!
ip prefix-list ALL seq 5 permit 0.0.0.0/0 le 32
!
ip prefix-list CLASSA seq 5 permit 0.0.0.0/1 le 27
!
ip prefix-list CLASSB seq 5 permit 128.0.0.0/2 ge 17
!         
ip prefix-list DEFAULTROUTE seq 5 permit 0.0.0.0/0
!
ip prefix-list FILTERTHIS seq 5 deny 172.16.1.0/24
ip prefix-list FILTERTHIS seq 10 permit 0.0.0.0/0 le 32
!
ip prefix-list RENETEST seq 5 permit 10.0.0.0/8 le 19
!
ip prefix-list RENETEST2 seq 5 permit 10.0.0.0/8 ge 20
!
ip prefix-list RENETEST3 seq 5 permit 10.0.0.0/8 ge 16 le 18
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface Loopback0
 ip address 172.16.0.1 255.255.255.0
!
interface Loopback1
 ip address 172.16.1.1 255.255.255.0
!
interface Loopback2
 ip address 172.16.2.1 255.255.255.0
!         
interface Loopback3
 ip address 172.16.3.1 255.255.255.0
!
interface Loopback10
 ip address 10.1.1.1 255.255.0.0
!
interface Loopback11
 ip address 10.2.2.2 255.255.128.0
!
interface Loopback12
 ip address 10.3.3.3 255.255.192.0
!
interface Loopback13
 ip address 10.4.4.4 255.255.224.0
!
interface Loopback14
 ip address 10.5.5.5 255.255.240.0
!
interface Loopback15
 ip address 10.6.6.6 255.255.248.0
!
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
router eigrp 12
 network 10.0.0.0
 network 172.16.0.0 0.0.3.255
 network 192.168.12.0
!
end
```
{% endtab %}
{% endtabs %}

I hope you now have a better understanding of prefix-lists. Don’t just read this lesson and forget about it. It’s best to boot up your own routers and configure some prefix-lists. If you have any more questions, please leave a comment!
