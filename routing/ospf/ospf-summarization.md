# OSPF Summarization

If you are studying OSPF you will learn that OSPF uses LSA type 3 for inter-area routers and LSA type 5 for external prefixes that are redistributed into OSPF.

OSPF can do summarization, but it’s impossible to summarize within an area. This means we have to configure summarization on an ABR or ASBR. OSPF can only summarize our LSA types 3 and 5.

If you want summarization for OSPF, you will have to configure it yourself. I will show you how to do this for inter-area and external prefixes. Let’s start with an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-summarization-lsa-type-3.png" alt=""><figcaption></figcaption></figure>

Look at the topology above. If we don’t use summarization (which is the default), there will be an LSA for every specific prefix. If we have a link failure in area 1, then R1 (our ABR) will flood a new type 3 summary LSA, and this change has to be propagated throughout all our OSPF areas. Since the LSDB will change our OSPF routers, they will have to re-run the SPF algorithm, which takes time and CPU power.

If we use summarization, things will be different. I can create a summary on R1 to summarize the different type 3 summary LSAs. Instead of sending an LSA for 4.4.4.0 /24 and 4.5.5.0 /24, I could send 4.0.0.0 /8 or something alike.

If a link failure occurs in area 1, nothing will change for area 0 and area 2 since they don’t have the specific 4.4.4.0 /24 prefix in their LSDB but the 4.0.0.0 /8 summary. Nothing will change in their LSDB, so we don’t have to re-run the SPF algorithm.

Summarization of type 3 summary LSAs means we are creating a summary of all the interarea routes. This is why we call it interarea route summarization. There are a couple of things to be aware of:

* A summary route will only be advertised if you have at least one subnet that falls within the summary range.
* A summary route will have the cost of the subnet with the lowest cost that falls within the summary range.
* Your ABR that creates the summary route will create a null0 interface to prevent loops.
* OSPF is a classless routing protocol, so you can pick any subnet mask you like for prefixes.

If you look at my picture, you can see that 4.4.4.0 /24 and 4.5.5.0 /24 both fall within the 4.0.0.0 /8 summary. We will still advertise the summary if we have a link failure for the 4.4.4.0 /24 prefix. If 4.5.5.0 /24 fails as well, then the summary will be withdrawn since no subnet is left within the 4.0.0.0 /8 range.

Enough theory. Let’s take a look at the configuration of inter-area summarization. This is the topology we’ll use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-summarization-inter-area.png" alt=""><figcaption></figcaption></figure>

I’m going to show you an example of interarea route summarization. I’m going to use routers R1 and R2. R1 will have 4 loopback interfaces which are in area 0. The link between R1 and R2 is in area 1, which turns R1 into an ABR that can do summarization.

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 172.16.0.0 0.0.3.255 area 0
</strong><strong>R1(config-router)#network 192.168.12.0 0.0.0.255 area 1
</strong></code></pre>

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 192.168.12.0 0.0.0.255 area 1
</strong></code></pre>

Here are the network commands I used to advertise all subnets.

<pre><code><strong>R2#show ip route ospf 
</strong>     172.16.0.0/32 is subnetted, 4 subnets
O IA    172.16.1.1 [110/2] via 192.168.12.1, 00:08:04, FastEthernet0/0
O IA    172.16.0.1 [110/2] via 192.168.12.1, 00:08:04, FastEthernet0/0
O IA    172.16.3.1 [110/2] via 192.168.12.1, 00:08:04, FastEthernet0/0
O IA    172.16.2.1 [110/2] via 192.168.12.1, 00:08:04, FastEthernet0/0
</code></pre>

<pre><code><strong>R2#show ip ospf database | begin Summary
</strong>		Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
172.16.0.1      172.16.3.1      542         0x80000003 0x005269
172.16.1.1      172.16.3.1      542         0x80000003 0x004773
172.16.2.1      172.16.3.1      542         0x80000003 0x003C7D
172.16.3.1      172.16.3.1      542         0x80000003 0x003187
</code></pre>

Above, you see the LSDB and routing table of R2. You can see there are 4 LSAs for each of the prefixes.

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#area 0 range 172.16.0.0 255.255.0.0
</strong></code></pre>

Using the `area range` command, we can summarize the type 3 summary LSAs. In my example, I’m creating the summary 172.16.0.0 /16. To keep things interesting, you need to type in a subnet mask for the summary instead of a wildcard for the `network` command.

```
R2#show ip route ospf                   
O IA 172.16.0.0/16 [110/2] via 192.168.12.1, 00:03:26, FastEthernet0/0
```

<pre><code><strong>R2#show ip ospf database | begin Summary
</strong>		Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
172.16.0.0      172.16.3.1      219         0x80000001 0x00605E
</code></pre>

Once again, the LSDB and routing table of R2. Instead of 4x type 3 summary LSA, we now have just a single LSA. You can see only the 172.16.0.0 /16 entry in the routing table.

So far, so good? Excellent! External route summarization is another thing we can do with OSPF and summarization. This is where we summarize the type 5 external LSAs. Two things to keep in mind:

* You can create the summary only on the ASBR.
* A null0 entry will be created in the routing table for the summary route.

This is the topology I will use to demonstrate this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-summarization-external.png" alt=""><figcaption></figcaption></figure>

Instead of using the network command to advertise the prefixes on the loopback interfaces, I’m going to redistribute them into OSPF.

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#no network 172.16.0.0 0.0.3.255 area 0
</strong><strong>R1(config-router)#redistribute connected subnets
</strong></code></pre>

We’ll remove the network command for the interfaces and redistribute the loopback interfaces into OSPF.

<pre><code><strong>R2#show ip route ospf   
</strong>     172.16.0.0/24 is subnetted, 4 subnets
O E2    172.16.0.0 [110/20] via 192.168.12.1, 00:01:07, FastEthernet0/0
O E2    172.16.1.0 [110/20] via 192.168.12.1, 00:01:07, FastEthernet0/0
O E2    172.16.2.0 [110/20] via 192.168.12.1, 00:01:07, FastEthernet0/0
O E2    172.16.3.0 [110/20] via 192.168.12.1, 00:01:07, FastEthernet0/0
</code></pre>

<pre><code><strong>R2#show ip ospf database | begin Type-5
</strong>		Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
172.16.0.0      172.16.3.1      91          0x80000001 0x00B46E 0
172.16.1.0      172.16.3.1      91          0x80000001 0x00A978 0
172.16.2.0      172.16.3.1      91          0x80000001 0x009E82 0
172.16.3.0      172.16.3.1      91          0x80000001 0x00938C 0
</code></pre>

Here is the LSDB and routing table of R2. As you can see we have type 5 external LSAs that show up as O E2 entries in the routing table.

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#summary-address 172.16.0.0 255.255.0.0
</strong></code></pre>

This is how you summarize the type 5 external LSAs by using the summary-address command. This is a different command compared to summarizing the type 3 summary LSAs.

<pre><code><strong>R2#show ip route ospf                  
</strong>O E2 172.16.0.0/16 [110/20] via 192.168.12.1, 00:00:17, FastEthernet0/0
</code></pre>

<pre><code><strong>R2#show ip ospf database | begin Type-5
</strong>		Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
172.16.0.0      172.16.3.1      38          0x80000002 0x00B26F 0
</code></pre>

This is what the LSDB and routing table of R2 looks like after the summarization. That’s all I wanted to show you about OSPF summarization.

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
interface FastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 1
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
interface Loopback0
 ip address 3.3.3.1 255.255.255.0
!
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 1
!
end
```
{% endtab %}
{% endtabs %}

I hope this has been helpful to you. Leave a comment if you have any questions!
