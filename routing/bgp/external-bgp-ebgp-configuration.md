# External BGP (eBGP) Configuration

In this lesson I will show you how to configure EBGP (External BGP) and how to advertise networks. I will be using the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/bgp-as1-as2.png" alt=""><figcaption></figcaption></figure>

Let’s start with a simple topology. Just two routers and two autonomous systems. Each router has a network on a loopback interface, which we will advertise in BGP.

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong></code></pre>

Use the **`router bgp`** command with the AS number to start BGP. Neighbors are not configured automatically. This is something you’ll have to do yourself with the **`neighbor x.x.x.x remote-as`** command. This is how we configure **external BGP**.

```
R1# %BGP-5-ADJCHANGE: neighbor 192.168.12.2 Up
```

```
R2# %BGP-5-ADJCHANGE: neighbor 192.168.12.1 Up
```

If everything goes ok, you should see a message that we have a new BGP neighbor adjacency.

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 password MYPASS
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 password MYPASS
</strong></code></pre>

If you like, you can enable MD5 authentication by using the **`neighbor password`** command. Your router will calculate an MD5 digest of every TCP segment sent.

<pre><code><strong>R1#show ip bgp summary 
</strong>BGP router identifier 1.1.1.1, local AS number 1
BGP table version is 1, main routing table version 1

Neighbor     V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.2 4     2      10      10        1    0    0 00:07:12        0
</code></pre>

<pre><code><strong>R2#show ip bgp summary 
</strong>BGP router identifier 2.2.2.2, local AS number 2
BGP table version is 1, main routing table version 1

Neighbor     V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.1 4     1      11      11        1    0    0 00:08:33        0
</code></pre>

**`Show ip bgp summary`** is an excellent command to check if you have BGP neighbors. You also see how many prefixes you received from each neighbor.

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#network 2.2.2.0 mask 255.255.255.0
</strong></code></pre>

Let’s advertise the loopback interface by using the **`network`** command. If you want to advertise something with BGP, you need to type the **exact subnet mask** for the network you want to advertise. If I type `network 1.0.0.0 mask 255.0.0.0` on R1, it will not work since this entry is not in the routing table.

<pre><code><strong>R1#show ip bgp 
</strong>BGP table version is 3, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       0.0.0.0                  0         32768 i
*> 2.2.2.0/24       192.168.12.2             0             0 2 i
</code></pre>

Use **`show ip bgp`** to look at the **BGP database**. You can see that R1 has learned about network 2.2.2.0 /24, and the next hop IP address is 192.168.12.2. It also shows the **path** information. You can see that network 2.2.2.0 /24 is from AS 2. Let’s check R2:

<pre><code><strong>R2#show ip bgp 
</strong>BGP table version is 3, local router ID is 2.2.2.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.12.1             0             0 1 i
*> 2.2.2.0/24       0.0.0.0                  0         32768 i
</code></pre>

R2 learned about network 1.1.1.0/24 with a next hop IP address of 192.168.12.1. Let’s check the routing tables:

<pre><code><strong>R1#show ip route bgp
</strong>     2.0.0.0/24 is subnetted, 1 subnets
B       2.2.2.0 [20/0] via 192.168.12.2, 00:16:13
</code></pre>

<pre><code><strong>R2#show ip route bgp 
</strong>     1.0.0.0/24 is subnetted, 1 subnets
B       1.1.1.0 [20/0] via 192.168.12.1, 00:16:59
</code></pre>

In the routing table, we can find an entry for BGP with an administrative distance of 20 for external BGP.

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
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 neighbor 192.168.12.2 password MYPASS
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
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 neighbor 192.168.12.1 password MYPASS
 network 2.2.2.0 mask 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

That’s all for now! If you have any questions feel free to ask.
