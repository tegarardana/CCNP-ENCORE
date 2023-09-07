# Multicast PIM Bootstrap Router (BSR)

BSR (Bootstrap) is similar to [Cisco’s AutoRP](https://networklessons.com/multicast/multicast-ip-pim-auto-rp/), it’s a protocol that we use to automatically find the RP (Rendezvous Point) in our multicast network. BSR however, is a **standard** and included in PIMv2, unlike AutoRP which is a Cisco proprietary protocol.

BSR uses two roles:

* **Candidate BSR**: this is the router that collects information from all available RPs in the network and advertises is throughout the network. It’s function is similar to the mapping agent in AutoRP.
* **Candidate RP**: these are routers that are advertising themselves who want to become the RP.

## Candidate BSR

The BSR sends messages on a **hop-by-hop basis** and does so by sending its packets to multicast address 224.0.0.13 with a TTL of 1. These multicast packets have a local scope thanks to their TTL so they are not routed.When a multicast router receives the message, it will do an RPF check on the source address of the BSR and will **re-send** the message on all other PIM-enabled interfaces. This is a big difference with the mapping agent of AutoRP which uses multicast where we require dense mode flooding to flood the packets of the mapping agent throughout the network.

The BSR messages will contain information about the BSR and of course the RP-to-group mappings.

Here is an illustration of this process:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/05/bsr-bootstrap-router-topology.png" alt=""><figcaption></figcaption></figure>

Above we have a small network with six routers. R3 is the BSR and sending BSR messages on its interfaces. All other routers will re-send these messages which allows the entire network to learn the information in the BSR messages.

There can be only **one active BSR** in the network. BSR routers will listen for messages from other BSR routers, the BSR router with the **highest priority** will become the active BSR. The highest BSR priority is 255 (the higher the better). When the priority is the same then the router with the **highest IP address** will become the BSR.

## Candidate RP

Routers that want to become an RP are called candidate RPs and they have to announce themselves to the BSR. When a candidate RP receives a BSR message, it will learn the source address of the BSR and it will register itself using **unicast packets**. This is another difference with AutoRP where the candidate RPs announce themselves using multicast packets.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/05/multicast-pim-bootstrap-rp-announcement.png" alt=""><figcaption></figcaption></figure>

Above you can see two routers, R2 and R6 who would like to become the RP. They are sending their RP announcement packets to the unicast IP address of the BSR (3.3.3.3).

Once the BSR receives the RP announcements, it will build a list of all RPs and the multicast groups they want to serve. This is called the group-to-RP mapping set.

The BSR will then include this list in its BSR messages so that all multicast routers receive it. One big difference with the mapping agent of AutoRP is that the BSR router **does not select the RP to use**. It will advertise **all RPs** with the multicast groups that they want to serve, it’s up to the multicast router to select the RP it wants to use.

## RP Selection

The multicast routers will receive multiple group-to-rp mapping sets from the BSR and they’ll have to select the best RP from this list. When you have multiple RPs then it’s very likely that you have multiple RPs that want to serve the same multicast groups. Here’s how we select the RP:

1. We look for the **longest match** on **a multicast group**. For example, let’s say we want to join multicast group 239.1.1.1. One RP advertises itself that it serves the 239.0.0.0/8 group range and another RP wants to serve the 239.1.1.0/24 range. In this case, we will prefer the second RP since 239.1.1.0/24 is the longest match.
2. When two RPs advertise the exact same groups then we will prefer the RP with the **highest priority**. Unlike the BSR selection, highest priority for RP selection means the **lowest priority value** (0 is the highest priority). The priority is a value we can configure on the RP.
3. If the groups and priority on the RP are the same then we will look at the **highest** **hash value** from a **hash function** to decide what the best RP is. I’ll explain the hash function in a bit.
4. If the groups, priority, and hash value of all RPs is the same then the RP with the **highest IP address** will be preferred.

### Hash Function

The hash function is used by **all multicast routers** to map multicast group addresses to different RPs.

for the sake of completeness, this is what the hash function looks like:

```
Value(G,M,C(i))=
   (1103515245 * ((1103515245 * (G&M)+12345) XOR C(i)) + 12345) mod 2^31
```

The input for this function is:

* **G**: the multicast group address.
* **M**: a hash mask value.
* **C**: the IP address of the RP.

Don’t get scared by the formula above, the router will do the calculations for you 🙂

The hash mask is a 32-bit value and **advertised in the BSR messages**. The router takes these values, puts them in the hash function and returns a hash value. The RP with the **highest hash value** will be selected.

By default, the **hash mask value is 0** which means that the hash value is only calculated based on the IP address of the RP.

Things become interesting when we change the hash mask. The remaining bits in the hash mask decide how many groups we map to an RP.

The hash mask is a 32-bit value so if you use a 31-bit mask, we only have one bit left. With a single bit, we can create two values which means that two multicast groups will be mapped to one RP.

If you use a 30-bit hash mask, we have two bits left. With two bits, we can create four values which means that four multicast groups will be mapped to one RP.

Let me give you an example:

We have two RPs with the same priority, both want to become the RP for the entire 239.0.0.0/8 multicast range. We have multicast listeners who want to receive traffic for the following multicast groups:

* 239.0.0.0
* 239.0.0.1
* 239.0.0.2
* 239.0.0.3
* 239.0.0.4
* 239.0.0.5
* 239.0.0.6
* 239.0.0.7

Without a hash mask, all eight multicast groups will be assigned to **one RP**.

With a 31-bit hash mask, we assign two multicast groups to each RP:

RP1:

* 239.0.0.0
* 239.0.0.1

RP2:

* 239.0.0.2
* 239.0.0.3

RP1:

* 239.0.0.4
* 239.0.0.5

RP2:

* 239.0.0.6
* 239.0.0.7

If you use a 30-bit hash mask, we assign four multicast groups to each RP:

RP1:

* 239.0.0.0
* 239.0.0.1
* 239.0.0.2
* 239.0.0.3

RP2:

* 239.0.0.4
* 239.0.0.5
* 239.0.0.6
* 239.0.0.7

This is a nice way to configure load balancing between RPs. The hash mask that you choose decides how many groups you map to one RP.

## Configuration

You have now learned the basics of BSR. Let’s see it in action! I will use the following topology for this example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/05/multicast-pim-bsr-topology.png" alt=""><figcaption></figcaption></figure>

First, we enable multicast routing on all routers::

<pre><code>R1, R2, R3 &#x26; R4
<strong>(config)#ip multicast-routing
</strong></code></pre>

And enable PIM sparse mode on all interfaces:

<pre><code><strong>R1(config)#interface GigabitEthernet 0/1
</strong><strong>R1(config-if)#ip pim sparse-mode
</strong></code></pre>

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#ip pim sparse-mode 
</strong>
<strong>R2(config)#interface GigabitEthernet 0/2
</strong><strong>R2(config-if)#ip pim sparse-mode 
</strong></code></pre>

<pre><code><strong>R3(config)#interface GigabitEthernet 0/1
</strong><strong>R3(config-if)#ip pim sparse-mode 
</strong>
<strong>R3(config)#interface GigabitEthernet 0/2
</strong><strong>R3(config-if)#ip pim sparse-mode 
</strong></code></pre>

<pre><code><strong>R4(config)#interface GigabitEthernet 0/1
</strong><strong>R4(config-if)#ip pim sparse-mode
</strong></code></pre>

R1 will become the RP and instead of using one of the physical interfaces, we will use a new loopback interface as the RP address. Make sure PIM sparse mode is enabled on the loopback and that it is advertised in OSPF:

<pre><code><strong>R1(config)#interface Loopback 0
</strong><strong>R1(config-if)#ip address 1.1.1.1 255.255.255.255
</strong><strong>R1(config-if)#ip pim sparse-mode
</strong>
<strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 1.1.1.1 0.0.0.0 area 0
</strong></code></pre>

Here’s the command to enable the RP:

<pre><code><strong>R1(config)#ip pim rp-candidate loopback 0 ?
</strong>group-list group-list
interval RP candidate advertisement interval
priority RP candidate priority
</code></pre>

Above you can see a number of options. The group-list allows you to use an access-list to select the multicast groups that you want to be an RP for. The priority command is used to set the priority. Let’s forget about these parameters and keep it simple for now:

<pre><code><strong>R1(config)#ip pim rp-candidate loopback 0
</strong></code></pre>

At the moment, nothing will happen. Our RP requires messages from a BSR so it knows where it can register itself. Let’s configure R2 as the BSR.

Like the RP, I will use a new loopback interface as the BSR address:

<pre><code><strong>R2(config)#interface Loopback 0
</strong><strong>R2(config-if)#ip address 2.2.2.2 255.255.255.255
</strong><strong>R2(config-if)#ip pim sparse-mode
</strong>
<strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 2.2.2.2 0.0.0.0 area 0
</strong></code></pre>

Now we can use the BSR command:

<pre><code><strong>R2(config)#ip pim bsr-candidate loopback 0 ?
</strong>Hash Mask length for RP selection
</code></pre>

There are a couple of parameters we can choose from. The first one is the hash mask length and we can set the priority:

<pre><code><strong>R2(config)#ip pim bsr-candidate loopback 0 0 ? 
</strong>  &#x3C;0-255>  Priority value for candidate bootstrap router
  &#x3C;cr>
</code></pre>

It’s also possible to configure a filter so the BSR doesn’t accept all RPs:

<pre><code><strong>R2(config)#ip pim bsr-candidate loopback 0 0 0 ?
</strong>  accept-rp-candidate  BSR RP candidate filter
  &#x3C;cr>
</code></pre>

We will keep it simple…I’m not going to use any of the parameters for now:

<pre><code><strong>R2(config)#ip pim bsr-candidate loopback 0
</strong></code></pre>

As soon as you configure the BSR, it will send BSR messages. Here’s what these look like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-pim-bootstrap-capture.png" alt=""><figcaption></figcaption></figure>

Above you can see that these packets are sent to multicast address 224.0.0.13. The BSR priority is 0 and the hash mask is 0.

Want to take a look at this packet yourself?

[Multicast PIM Bootstrap Capture](https://www.cloudshark.org/captures/3a1c8a56ac26)

Once our RP receives the BSR message, it will register itself. Here’s what the packet looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/04/multicast-pim-bootstrap-rp-announcement-capture.png" alt=""><figcaption></figcaption></figure>

Above you can see that this is a unicast packet. In the PIM options we see the default priority (0) and the multicast group: 224.0.0.0/4.

Want to see this packet for yourself?

[Multicast PIM Bootstrap RP Candidate Capture](https://www.cloudshark.org/captures/7dc1c53250e8)

Let’s verify our work:

<pre><code><strong>R1#show ip pim bsr-router 
</strong>PIMv2 Bootstrap information
  BSR address: 2.2.2.2 (?)
  Uptime:      00:01:04, BSR Priority: 0, Hash mask length: 0
  Expires:     00:02:06
  Candidate RP: 1.1.1.1(Loopback0)
    Holdtime 150 seconds
    Advertisement interval 60 seconds
    Next advertisement in 00:00:48
</code></pre>

The **show ip pim bsr-router** command gives us a quick overview of the BSR that our routers have found. Let’s check it on all routers:

<pre><code><strong>R2#show ip pim bsr-router
</strong>PIMv2 Bootstrap information
This system is the Bootstrap Router (BSR)
  BSR address: 2.2.2.2 (?)
  Uptime:      00:02:09, BSR Priority: 0, Hash mask length: 0
  Next bootstrap message in 00:00:51
</code></pre>

R2 sees itself as the BSR, so far so good. What about R3 and R4?

<pre><code><strong>R3#show ip pim bsr-router
</strong>PIMv2 Bootstrap information
  BSR address: 2.2.2.2 (?)
  Uptime:      00:01:11, BSR Priority: 0, Hash mask length: 0
  Expires:     00:01:59
</code></pre>

<pre><code><strong>R4#show ip pim bsr-router
</strong>PIMv2 Bootstrap information
  BSR address: 2.2.2.2 (?)
  Uptime:      00:01:13, BSR Priority: 0, Hash mask length: 0
  Expires:     00:01:57
</code></pre>

R3 and R4 also know where the BSR is. Let’s see if our routers also have learned where the RP is:

<pre><code><strong>R1#show ip pim rp mapping 
</strong>PIM Group-to-RP Mappings
This system is a candidate RP (v2)

Group(s) 224.0.0.0/4
  RP 1.1.1.1 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:04:35, expires: 00:01:54
</code></pre>

R1 knows that it’s a candidate RP and that this information comes from BSR. Let’s check R2:

<pre><code><strong>R2#show ip pim rp mapping
</strong>PIM Group-to-RP Mappings
This system is the Bootstrap Router (v2)

Group(s) 224.0.0.0/4
  RP 1.1.1.1 (?), v2
    Info source: 1.1.1.1 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:04:37, expires: 00:01:46
</code></pre>

R2 tells us that it’s the BSR and that it has learned that R1 is the RP. What about R3?

<pre><code><strong>R3#show ip pim rp mapping
</strong>PIM Group-to-RP Mappings

Group(s) 224.0.0.0/4
  RP 1.1.1.1 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:04:40, expires: 00:01:48
</code></pre>

R3 has learned about the RP, so does R4:

<pre><code><strong>R4#show ip pim rp mapping
</strong>PIM Group-to-RP Mappings

Group(s) 224.0.0.0/4
  RP 1.1.1.1 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:04:41, expires: 00:01:44
</code></pre>

All routers have now learned about the RP through BSR!

There are a couple of things we can try though. Let’s see what happens when we add a second RP. Let’s configure R4 as our second RP:

<pre><code><strong>R4(config)#interface loopback 0
</strong><strong>R4(config-if)#ip address 4.4.4.4 255.255.255.255
</strong><strong>R4(config-if)#ip pim sparse-mode 
</strong></code></pre>

<pre><code><strong>R4(config)#router ospf 1
</strong><strong>R4(config-router)#network 4.4.4.4 0.0.0.0 area 0
</strong>
<strong>R4(config)#ip pim rp-candidate loopback 0
</strong></code></pre>

Let’s check what our routers think of this:

<pre><code><strong>R1#show ip pim rp mapping
</strong>PIM Group-to-RP Mappings
This system is a candidate RP (v2)

Group(s) 224.0.0.0/4
  RP 4.4.4.4 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:02:23, expires: 00:02:20
  RP 1.1.1.1 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:18:42, expires: 00:02:04
</code></pre>

<pre><code><strong>R2#show ip pim rp mapping
</strong>PIM Group-to-RP Mappings
This system is the Bootstrap Router (v2)

Group(s) 224.0.0.0/4
  RP 4.4.4.4 (?), v2
    Info source: 4.4.4.4 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:02:21, expires: 00:02:05
  RP 1.1.1.1 (?), v2
    Info source: 1.1.1.1 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:18:29, expires: 00:01:43
</code></pre>

<pre><code><strong>R3#show ip pim rp mapping
</strong>PIM Group-to-RP Mappings

Group(s) 224.0.0.0/4
  RP 4.4.4.4 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:02:21, expires: 00:02:21
  RP 1.1.1.1 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:18:31, expires: 00:02:01
</code></pre>

<pre><code><strong>R4#show ip pim rp mapping
</strong>PIM Group-to-RP Mappings
This system is a candidate RP (v2)

Group(s) 224.0.0.0/4
  RP 4.4.4.4 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:02:21, expires: 00:02:23
  RP 1.1.1.1 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:18:29, expires: 00:02:04
</code></pre>

As you can see above, all routers know about both RPs. Both RPs advertise the 224.0.0.0/4 multicast group and have a priority of 0.

Let’s check the hash value of these RPs since that’s what decides which router we will use as the RP:

<pre><code><strong>R3#show ip pim rp-hash 239.1.1.1
</strong>  RP 4.4.4.4 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:11:38, expires: 00:02:09
  PIMv2 Hash Value (mask 0.0.0.0)
    RP 4.4.4.4, via bootstrap, priority 0, hash value 1642267698
    RP 1.1.1.1, via bootstrap, priority 0, hash value 332477713
</code></pre>

Above you can see that R4 has a higher hash value so that’s the RP we will use for multicast group 239.1.1.1. With a hash mask of 0, we will use R4 as the RP for all multicast groups.

Let’s try playing with the hash mask…we can change its value so that R1 and R4 will both be used as an RP. I will use a hash mask of 31-bit. This means that each RP will get two multicast groups:

<pre><code><strong>R2(config)#ip pim bsr-candidate loopback 0 31
</strong></code></pre>

Let’s use the show ip pim rp-hash command again to check different multicast groups:

<pre><code><strong>R3#show ip pim rp-hash 239.1.1.0
</strong>  RP 1.1.1.1 (?), v2
    Info source: 1.1.1.1 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 1w0d, expires: 00:01:46
  PIMv2 Hash Value (mask 255.255.255.254)
    RP 4.4.4.4, via bootstrap, priority 0, hash value 192087346
    RP 1.1.1.1, via bootstrap, priority 0, hash value 1415431185
</code></pre>

<pre><code><strong>R3#show ip pim rp-hash 239.1.1.1
</strong>  RP 1.1.1.1 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:41:36, expires: 00:01:48
  PIMv2 Hash Value (mask 255.255.255.254)
    RP 4.4.4.4, via bootstrap, priority 0, hash value 192087346
    RP 1.1.1.1, via bootstrap, priority 0, hash value 1415431185
</code></pre>

<pre><code><strong>R3#show ip pim rp-hash 239.1.1.2
</strong>  RP 4.4.4.4 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:26:07, expires: 00:01:28
  PIMv2 Hash Value (mask 255.255.255.254)
    RP 4.4.4.4, via bootstrap, priority 0, hash value 724279812
    RP 1.1.1.1, via bootstrap, priority 0, hash value 52024035
</code></pre>

<pre><code><strong>R3#show ip pim rp-hash 239.1.1.3
</strong>  RP 4.4.4.4 (?), v2
    Info source: 2.2.2.2 (?), via bootstrap, priority 0, holdtime 150
         Uptime: 00:26:35, expires: 00:01:59
  PIMv2 Hash Value (mask 255.255.255.254)
    RP 4.4.4.4, via bootstrap, priority 0, hash value 724279812
    RP 1.1.1.1, via bootstrap, priority 0, hash value 52024035
</code></pre>

Above you can see that 239.1.1.0 and 239.1.1.1 are now served by R1 and R4 gets 239.1.1.2 and 239.1.1.3. Each RP gets two multicast groups.

Last but not least, let’s see what happens when we add a second BSR?

I’ll configure R3 as the BSR:

<pre><code><strong>R3(config)#interface loopback 0
</strong><strong>R3(config-if)#ip address 3.3.3.3 255.255.255.255
</strong><strong>R3(config-if)#ip pim sparse-mode 
</strong>
<strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#network 3.3.3.3 0.0.0.0 area 0
</strong>
<strong>R3(config)#ip pim bsr-candidate Loopback0 31 
</strong></code></pre>

I’m using the same configuration as on R2. Let’s find out which router is the active BSR:

<pre><code><strong>R2#show ip pim bsr-router 
</strong>PIMv2 Bootstrap information
  BSR address: 3.3.3.3 (?)
  Uptime:      00:03:33, BSR Priority: 0, Hash mask length: 31
  Expires:     00:01:46
This system is a candidate BSR
  Candidate BSR address: 2.2.2.2, priority: 0, hash mask length: 31
</code></pre>

Above you can see that R3 is now the active BSR. The priority is the same on both routers so it’s the highest IP address that is used as a tie-breaker. R3 becomes the BSR.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip multicast-routing 
ip cef
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 ip pim sparse-mode
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
 ip pim sparse-mode
!
router ospf 1
 network 1.1.1.1 0.0.0.0 area 0
 network 192.168.12.0 0.0.0.255 area 0
!
ip pim rp-candidate Loopback0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip multicast-routing 
ip cef
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
 ip pim sparse-mode
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 ip pim sparse-mode
!
interface GigabitEthernet0/2
 ip address 192.168.23.2 255.255.255.0
 ip pim sparse-mode
!
router ospf 1
 network 2.2.2.2 0.0.0.0 area 0
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.23.0 0.0.0.255 area 0
!
ip pim bsr-candidate Loopback0 31
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ip multicast-routing 
ip cef
!
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
 ip pim sparse-mode
!
interface GigabitEthernet0/1
 ip address 192.168.34.3 255.255.255.0
 ip pim sparse-mode
!
interface GigabitEthernet0/2
 ip address 192.168.23.3 255.255.255.0
 ip pim sparse-mode
!
router ospf 1
 network 3.3.3.3 0.0.0.0 area 0
 network 192.168.23.0 0.0.0.255 area 0
 network 192.168.34.0 0.0.0.255 area 0
!
ip pim bsr-candidate Loopback0 31
!
end
```
{% endtab %}

{% tab title="R4" %}
```
hostname R4
!
ip multicast-routing 
ip cef
!
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
 ip pim sparse-mode
!
interface GigabitEthernet0/1
 ip address 192.168.34.4 255.255.255.0
 ip pim sparse-mode
!
router ospf 1
 network 4.4.4.4 0.0.0.0 area 0
 network 192.168.34.0 0.0.0.255 area 0
!
ip pim rp-candidate Loopback0
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

You have now learned the basics of BSR (Bootstrap) router. Let me summarize what we have seen here:

* BSR (Bootstrap) is a standard and integrated in PIMv2, it’s an alternative to Cisco’s AutoRP.
* BSR uses hop-by-hop messages and doesn’t require dense mode flooding.
* Each multicast router listens for BSR messages and forwards them on all PIM-enabled interfaces.
* The BSR router with the highest priority (255 is highest) becomes the active BSR. When the priority is the same then the highest IP address is used as a tie-breaker.
* Candidate RPs will register themselves (unicast) with the BSR once they learn the IP address of the BSR through the BSR messages.
* BSRs build group-to-RP mappings and advertise the entire list to other multicast routers.
* RP selection is based on:
  * longest match for the group address. 239.1.1.1/32 is a longer match than 239.0.0.0/8.
  * RP with the highest priority (0 is highest – opposite of BSR selection).
  * RP with the highest hash value which is calculated with the hash function.
  * RP with the highest IP address.
* The hash function can be used to assign different groups to different RPs and is a great way to implement load balancing.
* The default hash mask is 0 which means that all multicast groups will be assigned to one RP.

I hope you enjoyed this lesson! If you have any questions, feel free to leave a comment.
