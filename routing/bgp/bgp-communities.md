# BGP Communities

A BGP community is bit of “extra information” that you can add to one of more prefixes which is advertised to BGP neighbors. This extra information can be used for things like traffic engineering or dynamic routing policies. There are **4 well known BGP communities** that you can use or you can pick a **numeric value** that you can use for your own policies.

Here are the 4 well known BGP communities:

* Internet: advertise the prefix to all BGP neighbors.
* [No-Advertise](https://networklessons.com/bgp/bgp-community-no-advertise): don’t advertise the prefix to any BGP neighbors.
* [No-Export](https://networklessons.com/bgp/bgp-community-no-export): don’t advertise the prefix to any eBGP neighbors.
* [Local-AS](https://networklessons.com/bgp/bgp-community-local-as): don’t advertise the prefix outside of the sub-AS (this one is used for [BGP confederations](https://networklessons.com/bgp/bgp-confederation-explained)).

Once you finish reading this lesson, click on one of the links above to learn more about these well known BGP communities. I explained each of them in a separate lesson.

Why do we call them communities? A community is a group of prefixes that should be treated the same way. For example maybe you have 100 prefixes that require the same [local preference](https://networklessons.com/cisco/ccnp-encor-350-401/how-to-configure-bgp-local-preference-attribute) or [weight](https://networklessons.com/cisco/ccnp-encor-350-401/how-to-configure-bgp-weight-attribute). You could match all prefixes using an access-list or prefix-list but using BGP communities is more convenient.

Instead of manually selecting the prefixes, an ISP could instruct its customers to tag prefixes with a certain BGP community. When the customer does this, their prefixes get a certain treatment.

To give you an idea, here are some examples that I found from Level 3 (large ISP in the US):

```
--------------------------------------------------------
customer traffic engineering communities - Prepending
--------------------------------------------------------
65001:0   - prepend once  to all peers
65001:XXX - prepend once  at peerings to AS XXX
65002:0   - prepend twice to all peers
65002:XXX - prepend twice at peerings to AS XXX
65003:0   - prepend 3x to all peers
65003:XXX - prepend 3x    at peerings to AS XXX
65004:0   - prepend 4x to all peers
65004:XXX - prepend 4x    at peerings to AS XXX
--------------------------------------------------------
customer traffic engineering communities - Regional
--------------------------------------------------------
Will only work for regional peers
64980:0   - announce to customers but not to EU peers
64981:0   - prepend once  to all EU peers
64982:0   - prepend twice to all EU peers
64983:0   - prepend 3x    to all EU peers
64984:0   - prepend 4x    to all EU peers
--------------------------------------------------------
customer traffic engineering communities - LocalPref
--------------------------------------------------------
3356:70   - set local preference to 70
3356:80   - set local preference to 80
3356:90   - set local preference to 90
```

This list might not be up-to-date anymore but it gives you an impression of how BGP communities are used. If a customer of Level 3 tags their prefixes with 3356:90 then they will set the local preference to 90. If you tag them with 64983:0 then they will prepend the AS number three times to all their BGP neighbors in Europe.

These BGP communities are 32-bit values that are divided in two sections. For labs you can pick whatever values you like but normally the first 16 bits are used to indicate the AS number that originates the community, the next 16 bits are assigned by the AS. For example, Level 3 uses these communities:

```
--------------------------------------------------------
customer traffic engineering communities - LocalPref
--------------------------------------------------------
3356:70   - set local preference to 70
3356:80   - set local preference to 80
3356:90   - set local preference to 90
```

The first 16 bits is their AS number (3356) and the next 16 bits (70, 80 and 90) corresponds with the local preference value. On their routers they configured a policy that sets the local preference to these values if they receive prefixes with these BGP communities.

Nowadays we also use **extended communities** which are 8 octets. These are used often for MPLS VPN which we will discuss in another lesson. Let’s take a look at a configuration example so you can see how to implement BGP communities.

## Configuration

For this example I will use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/bgp-communities-as-path-prepend-example.png" alt=""><figcaption></figcaption></figure>

On the left side we have a customer router that is connected to ISP1. This ISP is connected to ISP2 and ISP3. Let’s imagine that ISP2 is somewhere in Europe and that ISP1 has a policy that they will prepend their AS number four times to BGP neighbors in Europe whenever a customer adds BGP community value 64984:0 to their prefixes.

Let’s see how we can configure this on the ISP1 and customer router.

### BGP Configuration

Here is the BGP configuration, it’s straight-forward eBGP:

<pre><code><strong>Customer#show running-config | section bgp
</strong>router bgp 10
 no synchronization
 bgp log-neighbor-changes
 network 10.10.10.10 mask 255.255.255.255
 neighbor 192.168.10.1 remote-as 1
 no auto-summary
</code></pre>

<pre><code><strong>ISP1#show running-config | section bgp
</strong>router bgp 1
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.10.10 remote-as 10
 neighbor 192.168.12.2 remote-as 2
 neighbor 192.168.13.3 remote-as 3
 no auto-summary
</code></pre>

<pre><code><strong>ISP2#show running-config | section bgp
</strong>router bgp 2
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.12.1 remote-as 1
 no auto-summary
</code></pre>

<pre><code><strong>ISP3#show running-config | section bgp
</strong>router bgp 3
 no synchronization
 bgp log-neighbor-changes
 neighbor 192.168.13.1 remote-as 1
 no auto-summary
</code></pre>

Let’s see if ISP1 has learned any prefixes from the customer router:

<pre><code><strong>ISP1#show ip bgp 10.10.10.10
</strong>BGP routing table entry for 10.10.10.10/32, version 2
Paths: (1 available, best #1, table Default-IP-Routing-Table)
Flag: 0x820
  Advertised to update-groups:
        1
  10
    192.168.10.10 from 192.168.10.10 (192.168.10.10)
      Origin IGP, metric 0, localpref 100, valid, external, best
</code></pre>

ISP1 has learned the network on the loopback interface of the customer router. Right now we don’t have any BGP communities. Let’s start with the configuration of ISP1…

### ISP1 AS Path Prepend Configuration

First we will create a community list that matches the community- value:

<pre><code><strong>ISP1(config)#ip community-list 1 permit 64984:0
</strong></code></pre>

The community-list is similar to an access-list or prefix-list but only used for BGP communities. Our next step is to create a route-map that will prepend the AS path whenever we see this value:

<pre><code><strong>ISP1(config)#route-map PREPEND_EU permit 10     
</strong><strong>ISP1(config-route-map)#match community 1
</strong><strong>ISP1(config-route-map)#set as prepend 1 1 1 1                 
</strong><strong>ISP1(config-route-map)#exit
</strong><strong>ISP1(config)#route-map PREPEND_EU permit 20
</strong></code></pre>

This route-map matches on community-list 1 and prepends the AS path four times. Let’s attach it outbound to ISP2:

<pre><code><strong>ISP1(config)#router bgp 1 
</strong><strong>ISP1(config-router)#neighbor 192.168.12.2 route-map PREPEND_EU out
</strong></code></pre>

This takes care of the configuration of ISP1. Let’s configure our customer router to send the BGP community with its prefix advertisement now…

### Customer BGP Community Configuration

On our customer router we will use a prefix-list to match the network on the loopback interface:

<pre><code><strong>Customer(config)#ip prefix-list LOOPBACK permit 10.10.10.10/32
</strong></code></pre>

And we’ll use a route-map to set the BGP community:

<pre><code><strong>Customer(config)#route-map SET_COMMUNITY permit 10
</strong><strong>Customer(config-route-map)#match ip address prefix-list LOOPBACK
</strong><strong>Customer(config-route-map)#set community 64984:0
</strong><strong>Customer(config-route-map)#exit
</strong><strong>Customer(config)#route-map SET_COMMUNITY permit 20
</strong></code></pre>

Everything that matches the prefix-list will have a community value of 64984:0. Now we have to activate this route-map:

<pre><code><strong>Customer(config)#router bgp 10
</strong><strong>Customer(config-router)#neighbor 192.168.10.1 route-map SET_COMMUNITY out
</strong><strong>Customer(config-router)#neighbor 192.168.10.1 send-community
</strong></code></pre>

Take a close look at the second command, we have to use the **neighbor send-community** command because the router doesn’t automatically send BGP communities to its neighbors. Everything is in place, let’s verify our work…

## Verification

To speed things up I will reset BGP:

<pre><code><strong>Customer#clear ip bgp *
</strong></code></pre>

Now take look at ISP1 to see if it received the BGP community:

<pre><code><strong>ISP1#show ip bgp 10.10.10.10
</strong>BGP routing table entry for 10.10.10.10/32, version 6
Paths: (1 available, best #1, table Default-IP-Routing-Table)
  Advertised to update-groups:
        1    2
  10
    192.168.10.10 from 192.168.10.10 (10.10.10.10)
      Origin IGP, metric 0, localpref 100, valid, external, best
      Community: 4258791424
</code></pre>

This looks interesting, it did receive our community value but it’s showing it as a big 32-bit decimal number. There’s a command on Cisco IOS that lets you choose between this output and the output with two 16-bit values. Let’s change it:

<pre><code><strong>ISP1(config)#ip bgp-community new-format
</strong></code></pre>

Use the **ip bgp community new-format** command and it will now look like this:

<pre><code><strong>ISP1#show ip bgp 10.10.10.10
</strong>BGP routing table entry for 10.10.10.10/32, version 6
Paths: (1 available, best #1, table Default-IP-Routing-Table)
  Advertised to update-groups:
        1    2
  10
    192.168.10.10 from 192.168.10.10 (10.10.10.10)
      Origin IGP, metric 0, localpref 100, valid, external, best
      Community: 64984:0
</code></pre>

That’s better…now let’s see if ISP1 has prepended the AS Path:

<pre><code><strong>ISP2#show ip bgp
</strong>BGP table version is 12, local router ID is 192.168.12.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 10.10.10.10/32   192.168.12.1                           0 1 1 1 1 1 10 i
</code></pre>

<pre><code><strong>ISP3#show ip bgp
</strong>BGP table version is 12, local router ID is 192.168.13.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 10.10.10.10/32   192.168.13.1                           0 1 10 i
</code></pre>

Great! Just like expected, the ISP1 router has prepended the AS path when it advertised the prefix to ISP2. Next time our customer wants something prepended, they only have to set the correct community value.

I hope this example has been useful to understand what BGP communities are about and how to implement them. Of course we still have our well known communities:

* Internet
* [No-Advertise](https://networklessons.com/bgp/bgp-community-no-advertise)
* [No-Export](https://networklessons.com/bgp/bgp-community-no-export)
* [Local-AS](https://networklessons.com/bgp/bgp-community-local-as)

Just click on the links above if you want to learn how these work. If you have any questions, feel free to leave a comment!
