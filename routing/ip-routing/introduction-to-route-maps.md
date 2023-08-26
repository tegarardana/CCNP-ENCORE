# Introduction to Route-Maps

Route-maps are the “if-then” programming solution for Cisco devices.  A route-map allows you to check for **certain match conditions** and (optionally) set a value.

Here are some quick examples:

* Only advertise some EIGRP routes to your neighbor.
  * Example: if prefix matches 192.168.1.0/24 in access-list then advertise it.
* Set [BGP attributes](https://networklessons.com/cisco/ccnp-encor-350-401/bgp-attributes-and-path-selection) based on certain match conditions.
  * Example: if prefix matches 192.168.0.0/24 then set the local preference to 500.
* [Redistribute networks](https://networklessons.com/cisco/ccnp-route/introduction-to-redistribution) from OSPF into EIGRP based on certain match conditions.
  * Example: if prefix matches 192.168.4.0/24 then redistribute it from OSPF into EIGRP.
* Change the next hop IP address with [policy-based routing](https://networklessons.com/cisco/ccnp-route/how-to-configure-policy-based-routing).
  * Example: if packet length > 500 bytes, change the next hop IP address to 192.168.1.254.

Route-maps are a bit like access-lists on steroids. They are far more powerful since besides prefixes, there are a lot of different match conditions and you set certain values.

In this lesson, I’ll give you a global overview of how route-maps work and I’ll show you how to configure them.

Like access-lists, route-maps work with different permit or deny statements:

<div align="left">

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/10/route-map-overview.png" alt=""><figcaption></figcaption></figure>

</div>

We start at the top and process the first statement. There are two possible outcomes:

* Match: there is a match, we apply our action and that’s it. We don’t check the other route-map statements to see if there is another match.
* No match: we continue and check the next route-map statement.

When you don’t have any matches, we hit the **invisible implicit deny** at the bottom of the route-map. This is similar to how an access-list works.

Each route-map can have one or more match conditions. Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/10/route-map-match-condition.png" alt=""><figcaption></figcaption></figure>

Our first two statements (10 and 20) have a match condition. There are a lot of possible match conditions. To name a few:

* prefix-list
* access-list
* BGP local preference
* BGP AS path
* Packet Length
* And many more…

If you don’t have a match condition then your statement **matches everything**.

Besides a match condition, we can also change something with a set command:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/10/route-map-set.png" alt=""><figcaption></figcaption></figure>

Route-map statements 10 and 30 have a set command. Here are some examples of set commands:

* Change the BGP AS path length.
* Set a BGP community.
* Set the BGP weight.
* Set the metric of an OSPF or EIGRP route in redistribution.
* Set a redistribution tag.
* Set the next hop IP address in policy-based routing.
* Set the DSCP value of an IP packet.
* And many other options…

This is the “if-then” logic of the route-map. IF we match a certain match condition, then SET something.

The best way to learn about route-maps is to see them in action.

## Configuration

To demonstrate route-maps, we need to create route-maps and have something to apply them to.  I’ll use two routers for this lesson:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/10/r1-r2-gigabit-interfaces.png" alt=""><figcaption></figcaption></figure>

EIGRP is pre-configured and R1 advertises some loopback interfaces to R2. We’ll use route-maps to filter networks that R1 advertises to R2.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip cef
!
interface Loopback0
 ip address 192.168.0.1 255.255.255.0
!
interface Loopback1
 ip address 192.168.1.1 255.255.255.0
!
interface Loopback2
 ip address 192.168.2.1 255.255.255.0
!
interface Loopback3
 ip address 192.168.3.1 255.255.255.0
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
router eigrp 1
 network 192.168.0.0 0.0.255.255
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
router eigrp 1
 network 192.168.0.0 0.0.255.255
!
end
```
{% endtab %}
{% endtabs %}

R2 has learned these four networks:

<pre><code><strong>R2#show ip route eigrp | include /24
</strong>D     192.168.0.0/24 
D     192.168.1.0/24 
D     192.168.2.0/24 
D     192.168.3.0/24
</code></pre>

Let’s see what we can do with route-maps.

### Match Condition- Permit

Let’s create a new route-map and see what options we have:

<pre><code><strong>R2(config)#route-map ?
</strong>WORD  Route map tag
</code></pre>

First, we need to give it a name. Let’s call it TEST\_1:

<pre><code><strong>R2(config)#route-map TEST_1 ?
</strong>  &#x3C;0-65535>  Sequence to insert to/delete from existing route-map entry
  deny       Route map denies set operations
  permit     Route map permits set operations
  
</code></pre>

I can choose between a _permit_ or _deny_ statement. So far, this is similar to how an access-list looks. Let’s go for _permit_ and use sequence number 10:

<pre><code><strong>R2(config)#route-map TEST_1 permit 10
</strong></code></pre>

Let’s look at the options of our route-map:

<pre><code><strong>R2(config-route-map)#?
</strong>    Route Map configuration commands:
      continue     Continue on a different entry within the route-map
      default      Set a command to its defaults
      description  Route-map comment
      exit         Exit from route-map configuration mode
      help         Description of the interactive help system
      match        Match values from routing table
      no           Negate a command or set its defaults
      set          Set values in destination routing protocol
</code></pre>

There are a couple of options to choose from. We’ll start with _match_:

<pre><code><strong>R2(config-route-map)#match ?
</strong>    additional-paths  BGP Add-Path match policies
    as-path           Match BGP AS path list
    clns              CLNS information
    community         Match BGP community list
    extcommunity      Match BGP/VPN extended community list
    interface         Match first hop interface of route
    ip                IP specific information
    ipv6              IPv6 specific information
    length            Packet length
    local-preference  Local preference for route
    mdt-group         Match routes corresponding to MDT group
    metric            Match metric of route
    mpls-label        Match routes which have MPLS labels
    policy-list       Match IP policy list
    route-type        Match route-type of route
    rpki              Match RPKI state of route
    security-group    Security Group
    source-protocol   Match source-protocol of route
    tag               Match tag of route
    track             tracking object
</code></pre>

Above, you see a big list of stuff you can match on. I want to use an access-list as my match condition. We can find this under the _ip_ parameter:

<pre><code><strong>R2(config-route-map)#match ip ?                      
</strong>    address                Match address of route or match packet
    flowspec               Match src/dest prefix component of flowspec prefix
    next-hop               Match next-hop address of route
    redistribution-source  route redistribution source (EIGRP only)
    route-source           Match advertising source address of route
</code></pre>

We have a couple of options. Let’s pick _address_:

<pre><code><strong>R2(config-route-map)#match ip address ?
</strong>    &#x3C;1-199>      IP access-list number
    &#x3C;1300-2699>  IP access-list number (expanded range)
    WORD         IP access-list name
    prefix-list  Match entries of prefix-lists
</code></pre>

Now I can choose between an access-list of prefix-list. Let’s refer to an access-list called “R1\_L0\_PERMIT”:

<pre><code><strong>R2(config-route-map)#match ip address R1_L0_PERMIT
</strong></code></pre>

We now have a route-map…great! It doesn’t do anything yet though, and we still need to create that access-list.

**Access-list Permit**

Let’s create the access-list that we refer to in our route-map. I’ll create a permit statement that matches network 192.168.0.0/24:

<pre><code><strong>R2(config)#ip access-list standard R1_L0_PERMIT 
</strong><strong>R2(config-std-nacl)#permit 192.168.0.0 0.0.0.255
</strong></code></pre>

The only thing left to do is to attach our route-map to something. We’ll keep it simple, I’ll attach it to a [distribute-list in EIGRP](https://networklessons.com/cisco/ccnp-route/eigrp-route-map-filtering). This allows us to filter networks that R1 advertises to R2:

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)#distribute-list route-map TEST_1 in
</strong></code></pre>

What I like about EIGRP is that it resyncs when you apply a distribute-list. This helps to speed things up when testing. You’ll see the following message on your console:

```
 %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.1 (GigabitEthernet0/1) is resync: route configuration changed
```

Right now, we have the following access-list and route-map:

```
ip access-list standard R1_L0_PERMIT
permit 192.168.0.0 0.0.0.255

route-map TEST_1 permit 10
 match ip address R1_L0_PERMIT
```

Let’s check the routing table of R2:

<pre><code><strong>R2#show ip route eigrp | include /24
</strong>D     192.168.0.0/24
</code></pre>

We only see the 192.168.0.0/24 network. What happened?

* Our route-map has a single permit statement that has our access-list as a match condition.
* Our access-list has a single permit statement for 192.168.0.0/24.
* Everything else is denied in the access-list by the invisible implicit deny any.
* We only have one route-map statement so we hit the invisible implicit deny any in the route-map.

Let’s continue with our next example.

#### **Access-list Deny**

Let’s try something else. Let’s create a route-map with a permit statement and an access-list with a deny statement:

<pre><code><strong>R2(config)#ip access-list standard R1_L0_DENY
</strong><strong>R2(config-std-nacl)#deny 192.168.0.0 0.0.0.255
</strong></code></pre>

Next, create a new route-map with this access-list:

<pre><code><strong>R2(config)#route-map TEST_2 permit 10
</strong><strong>R2(config-route-map)#match ip address R1_L0_DENY
</strong></code></pre>

Let’s apply the route-map to EIGRP:

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)#distribute-list route-map TEST_2 in
</strong></code></pre>

What do we have now? Let’s find out:

<pre><code><strong>R2#show ip route eigrp | include /24
</strong></code></pre>

We don’t have any routes. What happened?

* The route-map has one permit statement, with the access-list as the match condition.
* the access-list has a deny statement for 192.168.0.0 /24.
* There are no permit statements in the access-list so everything is denied by implicit deny any.
* We only have one route-map statement so we hit the implicit deny any in the route-map.

If your goal was to deny only 192.168.0.0/32 and permit everything else, then you should add a “permit any” to the access-list.

### Match Condition – Deny

Let’s reverse the logic of our route-map. In the previous two examples, I used permit statements. Now we’ll try a deny statement.

#### **Access-list Permit**

I’ll use the same access-list we created in the first example:

<pre><code><strong>R2#show access-lists R1_L0_PERMIT
</strong>Standard IP access list R1_L0_PERMIT
    10 permit 192.168.0.0, wildcard bits 0.0.0.255
</code></pre>

And we’ll create a route-map with a deny statement that uses the access-list as our match condition:

<pre><code><strong>R2(config)#route-map TEST_3 deny 10
</strong><strong>R2(config-route-map)#match ip address R1_L0_PERMIT
</strong></code></pre>

Let’s apply it to EIGRP:

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)#distribute-list route-map TEST_3 in
</strong></code></pre>

What do we have now? Let’s check the routing table of R2:

<pre><code><strong>R2#show ip route eigrp | include /24
</strong></code></pre>

There is nothing there…What happened?

* We have a single route-map statement that denies everything in the access-list.
* The access-list only matches 192.168.0.0/24. This means we have a match and we deny 192.168.0.0/24 from being advertised.
* Everything else hits the implicit deny any of our access-list.
* There are no other route-map statements so everything else hits the implicit deny any of the route-map.

What if I want to deny 192.168.0.0/24 but permit everything else?

If I want to fix this, I have to add an extra route-map statement:

<pre><code><strong>R2(config)#route-map TEST_3 permit 20
</strong></code></pre>

I don’t add any match rules in this sequence number. An empty route-map statement matches everything.

Here’s the result:

<pre><code><strong>R2#show ip route eigrp | include /24
</strong>D     192.168.1.0/24 
D     192.168.2.0/24 
D     192.168.3.0/24
</code></pre>

All other networks now show up. Let me show you the route-map and access-list one more time:

```
route-map TEST_3 deny 10
match ip address R1_L0_PERMIT

route-map TEST_3 permit 20
```

To wrap up this example, the first route-map statement denies the 192.168.0.0/24 network. The second route-map statement permits everything else.

#### **Access-list Deny**

This example usually confuses some people. I will use the access-list with the deny statement:

<pre><code><strong>R2#show access-lists R1_L0_DENY
</strong>Standard IP access list R1_L0_DENY
    10 deny   192.168.0.0, wildcard bits 0.0.0.255
</code></pre>

And let’s create a new route-map with a single deny statement:

<pre><code><strong>R2(config)#route-map TEST_4 deny 10
</strong><strong>R2(config-route-map)#match ip address R1_L0_DENY
</strong></code></pre>

Let’s apply it to EIGRP:

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)# distribute-list route-map TEST_4 in
</strong></code></pre>

What do we have in our routing table? Let’s find out:

<pre><code><strong>R2#show ip route eigrp | include /24
</strong></code></pre>

There’s nothing. What happened?

* Our route-map has a single deny statement. It denies everything in the access-list.
* The access-list denies 192.168.0.0/24. Everything else is denied as well by the implicit deny any any.
* Our first route-map statement doesn’t match anything so we continue with the next route-map statement.
* There is no next route-map statement, so everything is dropped by the implicit deny any in the route-map.

To prove that it works like this, we can add an empty route-map statement that permits everything:

<pre><code><strong>R2(config)#route-map TEST_4 permit 20
</strong></code></pre>

Let’s take another look at the routing table of R2:

<pre><code><strong>R2#show ip route eigrp | include /24
</strong>D     192.168.0.0/24 
D     192.168.1.0/24 
D     192.168.2.0/24 
D     192.168.3.0/24
</code></pre>

You have now seen how to match and permit/deny statements.

{% hint style="info" %}
What confuses some people is that they think the “double negative” (deny in route-map and deny in access-list) means that it will match. That’s not how a route-map works.
{% endhint %}

Let me give you a quick overview of what happens with the combinations of permit/deny statements that we have seen:

| Route-map | Match condition | Result                                                                                |
| --------- | --------------- | ------------------------------------------------------------------------------------- |
| permit    | match           | We have a match so we don’t process any other route-map statements.                   |
| permit    | no match        | No match so we continue with the next route-map statement to see if there is a match. |
| deny      | match           | We have a match so we don’t process any other route-map statements.                   |
| deny      | no match        | No match so we continue with the next route-map statement to see if there is a match. |

### Multiple Match Conditions

You now know how the permit and deny statements in route-maps work. Instead of a single match, we can also match multiple conditions.

Let’s create two new access-lists:

<pre><code><strong>R2(config)#ip access-list standard R1_L1_PERMIT
</strong><strong>R2(config-std-nacl)#permit 192.168.1.0 0.0.0.255
</strong>
<strong>R2(config)#ip access-list standard R1_L2_PERMIT
</strong><strong>R2(config-std-nacl)#permit 192.168.2.0 0.0.0.255
</strong></code></pre>

These access-lists match two of our loopback interfaces of R1. Let’s create a new route-map that matches both access-lists in a single statement:

<pre><code><strong>R2(config)#route-map MULTIPLE_MATCH permit 10
</strong><strong>R2(config-route-map)#match ip address R1_L1_PERMIT R1_L2_PERMIT
</strong></code></pre>

Let’s apply it to EIGRP:

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)#distribute-list route-map MULTIPLE_MATCH in
</strong></code></pre>

Let’s check the routing table of R2:

<pre><code><strong>R2#show ip route eigrp | include /24
</strong>D     192.168.1.0/24 
D     192.168.2.0/24
</code></pre>

We see both networks that we matched in the access-lists in the routing table.Before we continue, let’s get rid of that distribute-list:

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)#no distribute-list route-map MULTIPLE_MATCH in
</strong></code></pre>

### Set Action

What you have seen so far about route-maps is nice, but a route-map with only match statements is a bit of a glorified access-list.

The true power of route-maps is the **set command**.

To demonstrate this, we can’t use the distribute-list in EIGRP. I’ll use another example where we [redistribute](https://networklessons.com/cisco/ccnp-route/introduction-to-redistribution) a new loopback interface into EIGRP.

Let’s create a new loopback interface and an access-list that matches it:

<pre><code><strong>R1(config)#interface Loopback4
</strong><strong>R1(config-if)#ip address 172.16.1.1 255.255.255.0
</strong></code></pre>

<pre><code><strong>R1(config)#ip access-list standard R1_L4
</strong><strong>R1(config-std-nacl)#permit 172.16.1.0 0.0.0.255
</strong></code></pre>

Next, we’ll create a route-map with a single permit statement that uses the access-list as the match condition:

<pre><code><strong>R1(config)#route-map SET permit 10
</strong><strong>R1(config-route-map)#match ip address R1_L4
</strong></code></pre>

Let’s take a closer look at the set command:

<pre><code><strong>R1(config-route-map)#set ?
</strong>  aigp-metric       accumulated metric value
  as-path           Prepend string for a BGP AS-path attribute
  automatic-tag     Automatically compute TAG value
  clns              OSI summary address
  comm-list         set BGP community list (for deletion)
  community         BGP community attribute
  dampening         Set BGP route flap dampening parameters
  default           Set default information
  extcomm-list      Set BGP/VPN extended community list (for deletion)
  extcommunity      BGP extended community attribute
  global            Set to global routing table
  interface         Output interface
  ip                IP specific information
  ipv6              IPv6 specific information
  level             Where to import route
  lisp              Locator ID Separation Protocol specific information
  local-preference  BGP local preference path attribute
  metric            Metric value for destination routing protocol
  metric-type       Type of metric for destination routing protocol
  mpls-label        Set MPLS label for prefix
  origin            BGP origin code
  tag               Tag value for destination routing protocol
  traffic-index     BGP traffic classification number for accounting
  vrf               Define VRF name
  weight            BGP weight for routing table
</code></pre>

There are a lot of options here. Since we are redistributing into EIGRP, let’s use the set command to set the metric:

<pre><code><strong>R1(config-route-map)#set metric ?
</strong>  +/-     Add or subtract metric
  &#x3C;0-4294967295>  Metric value or Bandwidth in Kbits per second
</code></pre>

EIGRP uses a composite metric so we need to set the bandwidth, delay, reliability, load, and MTU:

<pre><code><strong>R1(config-route-map)#set metric 1500 10 255 255 1500
</strong></code></pre>

Now we can redistribute the loopback into EIGRP and refer to our route-map:

<pre><code><strong>R1(config)#router eigrp 1
</strong><strong>R1(config-router)#redistribute connected route-map SET
</strong></code></pre>

Let’s look at R2:

<pre><code><strong>R2#show ip route eigrp
</strong>
      172.16.0.0/24 is subnetted, 1 subnets
D EX     172.16.1.0 
           [170/1709312] via 192.168.12.1, 00:01:41, GigabitEthernet0/1
</code></pre>

Above, we see the redistributed route and its metric. We can verify that the route-map works by changing the metric values:

<pre><code><strong>R1(config)#route-map SET permit 10
</strong><strong>R1(config-route-map)#set metric 15000 1000 100 100 1500
</strong></code></pre>

Let’s check the routing table to see the new metric value:

<pre><code><strong>R2#show ip route eigrp | include 170
</strong>           [170/426752] via 192.168.12.1, 00:00:29, GigabitEthernet0/1
</code></pre>

The metric has changed. Great!

{% hint style="info" %}
Like the match condition, you can also set multiple set commands.
{% endhint %}

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
 ip address 192.168.0.1 255.255.255.0
!
interface Loopback1
 ip address 192.168.1.1 255.255.255.0
!
interface Loopback2
 ip address 192.168.2.1 255.255.255.0
!
interface Loopback3
 ip address 192.168.3.1 255.255.255.0
!
interface Loopback4
 ip address 172.16.1.1 255.255.255.0
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
router eigrp 1
 network 192.168.0.0 0.0.255.255
 redistribute connected route-map SET
!
ip access-list standard R1_L4
 permit 172.16.1.0 0.0.0.255
!
route-map SET permit 10
 match ip address R1_L4
 set metric 15000 1000 100 100 1500
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
router eigrp 1
 network 192.168.0.0 0.0.255.255
!
ip access-list standard R1_L0_DENY
 deny   192.168.0.0 0.0.0.255
ip access-list standard R1_L0_PERMIT
 permit 192.168.0.0 0.0.0.255
ip access-list standard R1_L1_PERMIT
 permit 192.168.1.0 0.0.0.255
ip access-list standard R1_L2_PERMIT
 permit 192.168.2.0 0.0.0.255
!
route-map MULTIPLE_MATCH permit 10
 match ip address R1_L1_PERMIT R1_L2_PERMIT
!
route-map TEST_1 permit 10
 match ip address R1_L0_PERMIT
!
route-map TEST_3 deny 10
 match ip address R1_L0_PERMIT
!
route-map TEST_3 permit 20
!
route-map TEST_2 permit 10
 match ip address R1_L0_DENY
!
route-map TEST_4 permit 10
 match ip address R1_L0_DENY 1 2 3
!
route-map TEST_4 permit 20
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

You have now learned the basics of route-maps:

* Route-maps are the “if-then” solution for Cisco devices.
* We check for a match condition, then optionally set a value.
* Used often for BGP attributes, policy-based routing, redistribution, setting DSCP values, filtering, etc.
* How route-maps behave with different permit and/or deny statements.

I hope you enjoyed this lesson. If you have any questions feel free to leave a comment!
