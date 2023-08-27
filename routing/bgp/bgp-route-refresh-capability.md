# BGP Route Refresh Capability

A long time ago there was no method to dynamically request a re-advertisement of the prefixes of one of your BGP neighbors. When you change your policy, somehow you have to compare all the prefixes from your BGP neighbor against your new policy.

To solve this problem, the [soft reconfiguration](https://networklessons.com/cisco/ccnp-encor-350-401/bgp-soft-reset-reconfiguration) method was created which stores an unmodified version of the prefixes from your BGP neighbor. This works but you’ll need additional memory since you are saving an additional table for each BGP neighbor. Since 2000 we also have the route refresh capability, simply said…your router will ask its BGP neighbor to re-send its prefixes.

Here are the 3 options that we have to refresh our BGP table when our policy changes:

* Hard reset
* [Soft reconfiguration](https://networklessons.com/cisco/ccnp-encor-350-401/bgp-soft-reset-reconfiguration)
* Route refresh capability

The hard reset is the most simple method (clear ip bgp command). It kills the TCP session with your BGP neighbor which forces it to restart and as a result you’ll receive all prefixes from your neighbor again. It works but will interrupt your network, not a good idea.

The **soft reconfiguration** will store everything that you receive from a BGP neighbor in a separate table before applying the policy. I explain this in my [soft reconfiguration lesson](https://networklessons.com/cisco/ccnp-encor-350-401/bgp-soft-reset-reconfiguration). This works but it’s not very efficient. Your router will store an entire table for each BGP neighbor with the unmodified prefixes, you’ll need extra memory.

**Route refresh capability** is the most preferred method…when you change your BGP policy you just send a message to your BGP neighbor and it will re-send you all its prefixes, there will be **no disruption** at all.

In this lesson, we’ll look at the route refresh capability, it’s described in [RFC 2918](http://tools.ietf.org/rfc/rfc2918.txt) and supported on most routers.

## Configuration

I will use two routers for this, R1 and R2. I have added two loopback interfaces on R1 so that we have something to advertise:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/07/AS1-AS2-R1-R2-BGP-external.png" alt=""><figcaption></figcaption></figure>

Let’s start with a default BGP configuration:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong><strong>R1(config-router)#network 1.1.1.1 mask 255.255.255.255
</strong><strong>R1(config-router)#network 11.11.11.11 mask 255.255.255.255
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong></code></pre>

Route refresh is enabled by default, you can verify this by using the following show command:

<pre><code><strong>R1#show ip bgp neighbors 192.168.12.2 | begin capabilities
</strong>  Neighbor capabilities:
    Route refresh: advertised and received(new)
</code></pre>

This router can do a route refresh for inbound prefixes (what you learn from you BGP neighbor) or outbound (the prefixes that you send to them). On my IOS 15.x router you see “(new)” which means this router supports the RFC 2918 version of route refresh. Some older IOS versions might show (“old & new”) which means they also support a version of route refresh that Cisco implemented before the RFC was created.

Let’s see if R2 learned those prefixes on the loopback interfaces:

<pre><code><strong>R2#show ip bgp
</strong>BGP table version is 3, local router ID is 192.168.12.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.1/32       192.168.12.1             0             0 1 i
*> 11.11.11.11/32   192.168.12.1             0             0 1 i
</code></pre>

That’s looking good. Now I will create a route-map that changes one of the BGP attributes. This means the router will have to update its BGP table somehow:

<pre><code><strong>R2(config)#route-map METRIC permit 10
</strong><strong>R2(config-route-map)#set metric 222
</strong>
<strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 route-map METRIC in
</strong></code></pre>

This route-map will set the metric to 222 for all prefixes that we receive from R1. Let’s look at he BGP table again:

<pre><code><strong>R2#show ip bgp
</strong>BGP table version is 3, local router ID is 192.168.12.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.1/32       192.168.12.1             0             0 1 i
*> 11.11.11.11/32   192.168.12.1             0             0 1 i
</code></pre>

As you can see nothing has changed yet. We’ll use the route refresh method to fix this but before I do so, let’s enable a debug so you can see in realtime what is going on:

<pre><code><strong>R1 &#x26; R2#debug ip bgp
</strong>BGP debugging is on for address family: IPv4 Unicast
</code></pre>

I’ll enable the debug on both routers, now we can do a reset:

<pre><code><strong>R2#clear ip bgp 192.168.12.1 ?
</strong>  all              All address families
  flap-statistics  Clear flap statistic
  in               Soft reconfig inbound updates
  ipv4             Address family
  ipv6             Address family
  l2vpn            Address family
  nsap             Address family
  out              Soft reconfig outbound updates
  rtfilter         Address family
  slow             Forcefully clear slow-peer status and move it to original
                   update group
  soft             Soft reconfig inbound and outbound updates
  topology         Routing topology instance
  vpnv4            Address family
  vpnv6            Address family
  &#x3C;cr>
</code></pre>

You can choose between inbound, outbound or both. Let’s do inbound:

<pre><code><strong>R2#clear ip bgp 192.168.12.1 in
</strong></code></pre>

You will see the following message on R2:

```
R2#
BGP: 192.168.12.1 sending REFRESH_REQ(5) for afi/safi: 1/1
```

R2 is sending a refresh request to R1, let’s see what R1 thinks of this:

```
R1#
BGP: 192.168.12.2 rcvd REFRESH_REQ for afi/safi: 1/1
```

Now look at the BGP table of R2 again:

<pre><code><strong>R2#show ip bgp
</strong>BGP table version is 5, local router ID is 192.168.12.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.1/32       192.168.12.1           222             0 1 i
*> 11.11.11.11/32   192.168.12.1           222             0 1 i
</code></pre>

Very nice, the metric has been updated and we didn’t clear the BGP session…mission accomplished!

{% hint style="info" %}
When you enable soft reconfiguration, your router will no longer send a route refresh update request to its BGP neighbor but it will use the routing information that it stored for this neighbor.
{% endhint %}

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface Loopback 0
 ip address 1.1.1.1 255.255.255.255
!
interface Loopback 1
 ip address 11.11.11.11 255.255.255.255
!
interface fastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 network 1.1.1.1 mask 255.255.255.255
 network 11.11.11.11 mask 255.255.255.255
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
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 neighbor 192.168.12.1 route-map METRIC in
!
route-map METRIC permit 10
 set metric 222
!
end
```
{% endtab %}
{% endtabs %}

That’s all I have for now. Hopefully this has helped you to understand the route refresh capability. If you have any questions feel free to leave a comment.
