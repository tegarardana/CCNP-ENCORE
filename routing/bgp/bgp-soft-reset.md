# BGP Soft Reset

When we change the BGP routing policy (changing the attributes or adding filters) we need to reset the BGP session before the new policy takes effect. This is no problem in a lab but it’s something you don’t want to do in a production network. In fact, there are 3 methods how you can refresh your BGP policies:

* Hard reset
* Dynamic Soft Reset (route refresh)
* Soft reset with pre-stored information

The **hard reset** is the most simple method (clear ip bgp command). It kills the TCP session with your BGP neighbor which forces it to restart and as a result you’ll receive all prefixes from your neighbor again. It works, but it’s cruel…

**Dynamic soft reset** is the most preferred method, it requires the route refresh capability. Simply said, this feature lets your router request its BGP neighbor to send its prefixes again.

Routers that don’t support the route refresh capability will have to use the **soft reset** option. That’s what this lesson is about. You can read about dynamic soft reset / route refresh in my other lesson.

{% hint style="info" %}
Normally I talk about “prefixes” or “routes” but technically the information that BGP exchanges in update messages is called **NLRI (Network Layer Reachability Information)**. The NLRI field contains the prefixes and length.
{% endhint %}

The soft reset option uses “pre-stored” information. Basically when we receive prefixes from a BGP neighbor we will store this information in a new table and we don’t make any changes to it. Our router will then apply its inbound BGP policy to this table and stores the end result as the BGP table.

Since you are now storing another table for each neighbor instead of one BGP table you will have some overhead, your router will require more memory. This is especially true when you enable soft reset for all your BGP neighbors…keep this in mind before you configure this.

The tables that I’m talking about have some special names, let me show you a picture and explain this a bit more:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/07/adj-rib-in-loc-rib-adj-rib-out-1024x485.png" alt=""><figcaption></figcaption></figure>

On the left side we see a table called **adj-RIB-in**. This is the unedited routing information from a BGP neighbor. There’s a separate table for each BGP neighbor that you peer with. We apply our inbound BGP policy to this information and the result is a table called the **loc-RIB**, this is the actual BGP table.

BGP will select the best path from the BGP table and the router will install this in the routing table. Also, the best paths can be advertised to other BGP neighbors. We can apply an outbound BGP policy to outbound updates and when this is done we have a table called **adj-RIB-out** (per neighbor)**.** The adj-RIB-in table is actually stored in memory for each neighbor, the adj-RIB-out table not.

Now you have an idea about the different tables and how soft reconfiguration works, let’s take a look at this on some BGP routers.

## Configuration

To demonstrate the soft reset we only need two routers. R1 has two loopback interfaces so that we have a couple of networks to advertise:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/07/AS1-AS2-R1-R2-BGP-external.png" alt=""><figcaption></figcaption></figure>

First we will configure BGP between the two routers:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong><strong>R1(config-router)#network 1.1.1.1 mask 255.255.255.255
</strong><strong>R1(config-router)#network 11.11.11.11 mask 255.255.255.255
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong></code></pre>

Nothing special here, we run EBGP and R1 advertises its two loopback interfaces. By default the soft reset option is disabled, let’s configure it on R2:

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 soft-reconfiguration inbound
</strong></code></pre>

The **soft-reconfiguration inbound** command tells R2 to save the routing information from R1 unmodified in the adj-RIB-in table. It will then apply the inbound BGP policy and store the information in the BGP table.

Let’s take a look at these tables, a good way to do this is by changing some of the BGP attributes. I’ll change the local preference for the prefixes we receive from R1:

<pre><code><strong>R2(config)#route-map LOCALPREF permit 10
</strong><strong>R2(config-route-map)#set local-preference 200
</strong></code></pre>

<pre><code><strong>R2(config-route-map)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 route-map LOCALPREF in
</strong></code></pre>

This will set the local preference to 200 for all incoming prefixes from R1. Instead of clearing the TCP session, we’ll do a soft reset:

<pre><code><strong>R2#clear ip bgp 192.168.12.1 soft in
</strong></code></pre>

Use the **soft in** parameter to do a soft reset. Now look at the BGP table first:

<pre><code><strong>R2#show ip bgp
</strong>BGP table version is 3, local router ID is 192.168.12.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.1/32       192.168.12.1             0    200      0 1 i
*> 11.11.11.11/32   192.168.12.1             0    200      0 1 i
</code></pre>

The BGP table (loc-RIB) was modified as expected, now take a look at the adj-RIB-in table:

<pre><code><strong>R2#show ip bgp neighbors 192.168.12.1 received-routes
</strong>BGP table version is 3, local router ID is 192.168.12.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  1.1.1.1/32       192.168.12.1             0             0 1 i
*  11.11.11.11/32   192.168.12.1             0             0 1 i

Total number of prefixes 2
</code></pre>

Above you see the raw routing information from R1 before we applied the inbound BGP policy. You can see that no changes were made to the local preference of my prefixes.

Another nice experiment is to filter some of the prefixes:

<pre><code><strong>R2(config)#access-list 1 permit host 1.1.1.1
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 distribute-list 1 in
</strong></code></pre>

I’ll use a distribute-list so that 11.11.11.11 /32 is not allowed anymore. Before I do another soft reset I’ll enable a debug, this allows you to see what the router is doing with the BGP updates:

<pre><code><strong>R2#debug ip bgp updates
</strong>BGP updates debugging is on for address family: IPv4 Unicast
</code></pre>

Let’s do the soft reset:

<pre><code><strong>R2#clear ip bgp 192.168.12.1 soft in
</strong></code></pre>

Here’s what you will see:

```
R2#
BGP(0): start inbound soft reconfiguration for
BGP(0): process 1.1.1.1/32, next hop 192.168.12.1, metric 0 from 192.168.12.1
BGP(0): process 11.11.11.11/32, next hop 192.168.12.1, metric 0 from 192.168.12.1
BGP(0): Prefix 11.11.11.11/32 rejected by inbound distribute/prefix-list.
BGP(0): update denied
BGP(0): complete inbound soft reconfiguration, ran for 0ms
```

The router starts the soft reconfiguration, rejects the 11.11.11.11 /32 prefix and completes the soft reconfiguration. Take a look at the BGP table:

<pre><code><strong>R2#show ip bgp
</strong>BGP table version is 4, local router ID is 192.168.12.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.1/32       192.168.12.1             0    200      0 1 i
</code></pre>

As expected it’s gone but you will still find it in the adj-RIB-in table:

<pre><code><strong>R2#show ip bgp neighbors 192.168.12.1 received-routes
</strong>BGP table version is 4, local router ID is 192.168.12.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  1.1.1.1/32       192.168.12.1             0             0 1 i
*  11.11.11.11/32   192.168.12.1             0             0 1 i

Total number of prefixes 2
</code></pre>

Those are two good examples that show the difference between the adj-RIB-in and Loc-RIB tables. Of course we can also view the adj-RIB-out table, I’ll show you an example of R1:

<pre><code><strong>R1#show ip bgp neighbors 192.168.12.2 advertised-routes
</strong>BGP table version is 5, local router ID is 192.168.12.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external, f RT-Filter
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.1/32       0.0.0.0                  0         32768 i
*> 11.11.11.11/32   0.0.0.0                  0         32768 i

Total number of prefixes 2
</code></pre>

Use the **`show ip bgp neighbors advertised-routes`** command to view the adj-RIB-out table. These are all the prefixes that you advertise to each BGP neighbor.

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
 neighbor 192.168.12.1 distribute-list 1 in
!
access-list 1 permit host 1.1.1.1
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
 neighbor 192.168.12.1 soft-reconfiguration inbound
 neighbor 192.168.12.1 route-map LOCALPREF in
!
route-map LOCALPREF permit 10
 set local-preference 200
!
end
```
{% endtab %}
{% endtabs %}

I hope this lesson has been helpful to understand the soft reconfiguration feature. Make sure you also take a look at my route refresh lesson! If you have any questions, feel free to leave a comment.
