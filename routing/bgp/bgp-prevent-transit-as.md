# BGP Prevent Transit AS

By default, BGP will advertise all prefixes to EBGP (External BGP) neighbors. This means that if you are multi-homed (connected to two or more ISPs), you might become a transit AS. Let me show you an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/05/R1-two-ISPs-3-loopback.png" alt=""><figcaption></figcaption></figure>

R1 is connected to ISP1 and ISP2, and each router is in a different AS (Autonomous System). Since R1 is multi-homed, the ISPs may use R1 to reach each other. To prevent this, we’ll have to ensure that R1 only advertises prefixes from its own autonomous system.

As far as I know, there are four methods how you can prevent becoming a transit AS:

* Filter-list with AS PATH access-list.
* No-Export Community.
* Prefix-list Filtering
* Distribute-list Filtering

Prefix-lists or distribute-lists will work, but it’s not scalable if you have thousands of prefixes in your BGP table. The filter-list and no-export community work very well since you only have to configure them once, and it will not matter if new prefixes show up. First, we’ll configure BGP on each router:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong><strong>R1(config-router)#neighbor 192.168.13.3 remote-as 3
</strong></code></pre>

<pre><code><strong>ISP1(config)#router bgp 2
</strong><strong>ISP1(config-router)#neighbor 192.168.12.1 remote-as 1
</strong></code></pre>

<pre><code><strong>ISP2(config)#router bgp 3
</strong><strong>ISP2(config-router)#neighbor 192.168.13.1 remote-as 1
</strong></code></pre>

The commands above will configure EBGP (External BGP) between R1 – ISP1 and R1 – ISP2. To make sure we have something to look at, I’ll advertise the loopback interfaces in BGP on each router:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

<pre><code><strong>ISP1(config)#router bgp 2
</strong><strong>ISP1(config-router)#network 2.2.2.0 mask 255.255.255.0
</strong></code></pre>

<pre><code><strong>ISP2(config)#router bgp 3
</strong><strong>ISP2(config-router)#network 3.3.3.0 mask 255.255.255.0
</strong></code></pre>

With the networks advertised, let’s take a look at the BGP table of ISP1 and ISP2 to see what they have learned:

<pre><code><strong>ISP1#show ip bgp 
</strong>BGP table version is 4, local router ID is 11.11.11.11
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.12.1             0             0 1 i
*> 2.2.2.0/24       0.0.0.0                  0         32768 i
*> 3.3.3.0/24       192.168.12.1                           0 1 3 i
</code></pre>

<pre><code><strong>ISP2#show ip bgp 
</strong>BGP table version is 4, local router ID is 33.33.33.33
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.13.1             0             0 1 i
*> 2.2.2.0/24       192.168.13.1                           0 1 2 i
*> 3.3.3.0/24       0.0.0.0                  0         32768 i
</code></pre>

The ISP routers have learned about each other networks, and they will use R1 as the next hop. We now have everything in place to play with the different filtering techniques.

## Filter-list with AS-PATH access-list

Using a filter-list with the AS-PATH access-list is probably the most convenient solution. It will ensure that you will always only advertise prefixes from your own autonomous system. Here’s how to do it:

<pre><code><strong>R1(config)#ip as-path access-list 1 permit ^$
</strong>
<strong>R1(config-router)#neighbor 192.168.12.2 filter-list 1 out
</strong><strong>R1(config-router)#neighbor 192.168.13.3 filter-list 1 out
</strong></code></pre>

The ^$ regular expression ensures that we will only advertise locally originated prefixes. The ^ is the start of the string and the $ is the end of the string. This means we have an empty string. With BGP, the only time you see a path with no AS-PATH is when you have a route that your router originated or a route from an IBGP neighbor. In other words, a locally originated route.

We’ll have to apply this filter to both ISPs.

{% hint style="warning" %}
Keep in mind that BGP is slow…if you are doing labs, it’s best to speed things up with **`clear ip bgp *`**
{% endhint %}

Let’s verify our configuration:

<pre><code><strong>R1#show ip bgp 
</strong>BGP table version is 4, local router ID is 22.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       0.0.0.0                  0         32768 i
*> 2.2.2.0/24       192.168.12.2             0             0 2 i
*> 3.3.3.0/24       192.168.13.3             0             0 3 i
</code></pre>

R1 still knows about the prefixes from the ISP routers. What about ISP1 and ISP2?

<pre><code><strong>ISP1#show ip bgp 
</strong>BGP table version is 7, local router ID is 11.11.11.11
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.12.1             0             0 1 i
*> 2.2.2.0/24       0.0.0.0                  0         32768 i
</code></pre>

<pre><code><strong>ISP2#show ip bgp 
</strong>BGP table version is 7, local router ID is 33.33.33.33
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.13.1             0             0 1 i
*> 3.3.3.0/24       0.0.0.0                  0         32768 i
</code></pre>

ISP1 and ISP2 only know about the 1.1.1.0 /24 network. Excellent, we are no longer a transit AS!

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="ISP1" %}
```
hostname ISP1
!
interface Loopback 0
 ip address 2.2.2.2 255.255.255.0
!
interface fastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 network 2.2.2.0 mask 255.255.255.0
!
end
```
{% endtab %}

{% tab title="ISP2" %}
```
hostname ISP2
!
interface Loopback 0
 ip address 3.3.3.3 255.255.255.0
!
interface fastEthernet0/0
 ip address 192.168.13.3 255.255.255.0
!
router bgp 3
 neighbor 192.168.13.1 remote-as 1
 network 3.3.3.0 mask 255.255.255.0
!
end
```
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
 ip address 192.168.13.1 255.255.255.0
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 neighbor 192.168.13.3 remote-as 3
 network 1.1.1.0 mask 255.255.255.0
 neighbor 192.168.12.2 filter-list 1 out
 neighbor 192.168.13.3 filter-list 1 out
!
ip as-path access-list 1 permit ^$
!
end
```
{% endtab %}
{% endtabs %}

On to the next method…

## No-Export Community

Using the no-export community will also work pretty well. We will configure R1 so that prefixes from the ISP routers will be tagged with the no-export community. This ensures that the prefixes from those routers will be known within AS 1 but won’t be advertised to other routers.

<pre><code><strong>R1(config)#route-map NO-EXPORT
</strong><strong>R1(config-route-map)#set community no-export
</strong>
<strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 route-map NO-EXPORT in
</strong><strong>R1(config-router)#neighbor 192.168.13.3 route-map NO-EXPORT in
</strong></code></pre>

{% hint style="info" %}
I’m only using one router in AS 1, if you have other routers and are running IBGP (Internal BGP) then don’t forget to send communities to those routers with the **`neighbor <ip> send-community`** command.
{% endhint %}

Let’s see what ISP1 and ISP2 think about our configuration:

<pre><code><strong>ISP1#show ip bgp 
</strong>BGP table version is 11, local router ID is 11.11.11.11
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.12.1             0             0 1 i
*> 2.2.2.0/24       0.0.0.0                  0         32768 i
</code></pre>

<pre><code><strong>ISP2#show ip bgp 
</strong>BGP table version is 11, local router ID is 33.33.33.33
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.13.1             0             0 1 i
*> 3.3.3.0/24       0.0.0.0                  0         32768 i
</code></pre>

They only know about network 1.1.1.0 /24.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="ISP1" %}
```
hostname ISP1
!
interface Loopback 0
ip address 2.2.2.2 255.255.255.0
!
interface fastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 network 2.2.2.0 mask 255.255.255.0
!
end
```
{% endtab %}

{% tab title="ISP2" %}
```
hostname ISP2
!
interface Loopback 0
ip address 3.3.3.3 255.255.255.0
!
interface fastEthernet0/0
 ip address 192.168.13.3 255.255.255.0
!
router bgp 3
 neighbor 192.168.13.1 remote-as 1
 network 3.3.3.0 mask 255.255.255.0
!
end
```
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
 ip address 192.168.13.1 255.255.255.0
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 neighbor 192.168.13.3 remote-as 3
 network 1.1.1.0 mask 255.255.255.0
 neighbor 192.168.12.2 route-map NO-EXPORT in
 neighbor 192.168.13.3 route-map NO-EXPORT in
!
route-map NO-EXPORT
 set community no-export
!
end
```
{% endtab %}
{% endtabs %}

Onto the next method!

## Prefix-List Filtering

Using a prefix-list we can determine what prefixes are advertised to our BGP neighbors. This works fine, but it’s not a good solution to prevent becoming a transit AS. Each time you add new prefixes, you’ll have to reconfigure the prefix-list. Anyway, let me show you how it works:

<pre><code><strong>R1(config)#ip prefix-list NO-TRANSIT permit 1.1.1.0/24
</strong>
<strong>R1(config-router)#neighbor 192.168.12.2 prefix-list NO-TRANSIT out 
</strong><strong>R1(config-router)#neighbor 192.168.13.3 prefix-list NO-TRANSIT out
</strong></code></pre>

The prefix-list above will only advertise 1.1.1.0 /24 to the ISP routers. Let’s verify the configuration:

<pre><code><strong>ISP1#show ip bgp 
</strong>BGP table version is 17, local router ID is 11.11.11.11
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.12.1             0             0 1 i
*> 2.2.2.0/24       0.0.0.0                  0         32768 i
</code></pre>

<pre><code><strong>ISP2#show ip bgp 
</strong>BGP table version is 17, local router ID is 33.33.33.33
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.13.1             0             0 1 i
*> 3.3.3.0/24       0.0.0.0                  0         32768 i
</code></pre>

The prefix-list is working as it should.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="ISP1" %}
```
hostname ISP1
!
interface Loopback 0
ip address 2.2.2.2 255.255.255.0
!
interface fastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 network 2.2.2.0 mask 255.255.255.0
!
end
```
{% endtab %}

{% tab title="ISP2" %}
```
hostname ISP2
!
interface Loopback 0
ip address 3.3.3.3 255.255.255.0
!
interface fastEthernet0/0
 ip address 192.168.13.3 255.255.255.0
!
router bgp 3
 neighbor 192.168.13.1 remote-as 1
 network 3.3.3.0 mask 255.255.255.0
!
end
```
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
 ip address 192.168.13.1 255.255.255.0
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 neighbor 192.168.13.3 remote-as 3
 network 1.1.1.0 mask 255.255.255.0
 neighbor 192.168.12.2 prefix-list NO-TRANSIT out 
 neighbor 192.168.13.3 prefix-list NO-TRANSIT out
!
 ip prefix-list NO-TRANSIT permit 1.1.1.0/24
!
end
```
{% endtab %}
{% endtabs %}

Onto the last exercise!

## Distribute-list Filtering

This method is similar to using the prefix-list, but this time we’ll use an access-list.

<pre><code><strong>R1(config)#ip access-list standard NO-TRANSIT
</strong><strong>R1(config-std-nacl)#permit 1.1.1.0 0.0.0.255
</strong>
<strong>R1(config-router)#neighbor 192.168.12.2 distribute-list NO-TRANSIT out
</strong><strong>R1(config-router)#neighbor 192.168.13.3 distribute-list NO-TRANSIT out
</strong></code></pre>

Time to check the ISPs:

<pre><code><strong>ISP1#show ip bgp 
</strong>BGP table version is 23, local router ID is 11.11.11.11
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.12.1             0             0 1 i
*> 2.2.2.0/24       0.0.0.0                  0         32768 i
</code></pre>

<pre><code><strong>ISP2#show ip bgp 
</strong>BGP table version is 23, local router ID is 33.33.33.33
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.1.1.0/24       192.168.13.1             0             0 1 i
*> 3.3.3.0/24       0.0.0.0                  0         32768 i
</code></pre>

That’s all there is to it.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="ISP1" %}
```
hostname ISP1
!
interface Loopback 0
ip address 2.2.2.2 255.255.255.0
!
interface fastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 network 2.2.2.0 mask 255.255.255.0
!
end
```
{% endtab %}

{% tab title="ISP2" %}
```
hostname ISP2
!
interface Loopback 0
ip address 3.3.3.3 255.255.255.0
!
interface fastEthernet0/0
 ip address 192.168.13.3 255.255.255.0
!
router bgp 3
 neighbor 192.168.13.1 remote-as 1
 network 3.3.3.0 mask 255.255.255.0
!
end
```
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
 ip address 192.168.13.1 255.255.255.0
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 neighbor 192.168.13.3 remote-as 3
 network 1.1.1.0 mask 255.255.255.0
 neighbor 192.168.12.2 distribute-list NO-TRANSIT out
 neighbor 192.168.13.3 distribute-list NO-TRANSIT out
!
ip access-list standard NO-TRANSIT
 permit 1.1.1.0 0.0.0.255
!
end
```
{% endtab %}
{% endtabs %}

I hope this has been helpful to you. If you know of any other methods to prevent becoming a BGP transit AS, please leave a comment!
