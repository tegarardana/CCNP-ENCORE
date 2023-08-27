# BGP AS-Path Filtering

In this lesson, we’ll take a look at BGP AS path filtering. Using the AS path filter we can permit or deny prefixes from certain autonomous systems. You can use this for things like:

* Accept only prefixes from directly connected autonomous systems
* Accept only prefixes from directly connected autonomous systems AND one autonomous system behind the first one.
* Deny certain transit autonomous systems
* And more

To create rules like the examples above we need a flexible way so that we can “match” on certain autonomous systems. This can be done with [regular expressions](https://networklessons.com/bgp/bgp-regular-expressions-examples/). If you have no clue how regular expressions work then please read my [regexp lesson first](https://networklessons.com/bgp/bgp-regular-expressions-examples/).

Having said that, let’s look at some examples. I will use a BGP looking glass server for this.

A looking glass server is a router on the Internet that has a (full) internet routing table. You can use telnet to one and use show commands to view the BGP table. It’s a great way to practice regular expressions since there’s plenty of prefixes to play with.

You can find a looking glass server on [BGP4.as](http://www.bgp4.as/looking-glasses), I picked one that is close to me:

**route-server.tinet.net**

Once I connect to it through telnet this is what I see:

```
+--------------------------------------------------------------------+
|                                                                    |
|                    GTT Route Monitor - AS3257                      |
|                                                                    |
|   This system is solely for internet operational purposes. Any     |
|   misuse is strictly prohibited. All connections to this router    |
|   are logged.                                                      |
|                                                                    |
|   This server provides a view on the Tinet legacy routing table    |
|   that is used in Frankfurt/Germany. If you are interested in      |
|   other regions of the backbone check out http://www.as3257.net/   |
|                                                                    |
|                Please report problems to noc@gtt.net               |
|                                                                    |
+--------------------------------------------------------------------+

route-server.as3257.net>
```

Let’s see what we find in the BGP table:

<pre><code><strong>route-server.as3257.net>show ip bgp
</strong>BGP table version is 4491321, local router ID is 213.200.87.253
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.0.0.0/24       213.200.64.93            0             0 3257 15169 i
*> 1.0.4.0/24       213.200.64.93            0             0 3257 6453 7545 56203 i
*> 1.0.5.0/24       213.200.64.93            0             0 3257 6453 7545 56203 i
*> 1.0.6.0/24       213.200.64.93            0             0 3257 174 4826 38803 56203 i
*> 1.0.7.0/24       213.200.64.93            0             0 3257 174 4826 38803 56203 i
*> 1.0.20.0/23      213.200.64.93         1551             0 3257 2516 2519 i
*> 1.0.22.0/23      213.200.64.93         1551             0 3257 2516 2519 i
*> 1.0.24.0/23      213.200.64.93         1551             0 3257 2516 2519 i
*> 1.0.26.0/23      213.200.64.93         1551             0 3257 2516 2519 i
*> 1.0.28.0/22      213.200.64.93         1551             0 3257 2516 2519 i
*> 1.0.38.0/24      213.200.64.93          815             0 3257 9304 24155 i
*> 1.0.41.0/24      213.200.64.93          815             0 3257 9304 24155 i
*> 1.0.43.0/24      213.200.64.93          815             0 3257 9304 24155 i
*> 1.0.46.0/24      213.200.64.93          815             0 3257 9304 24155 i
*> 1.0.48.0/24      213.200.64.93          815             0 3257 9304 24155 i
*> 1.0.64.0/18      213.200.64.93         1551             0 3257 2516 7670 18144 i
*> 1.0.128.0/18     213.200.64.93            0             0 3257 174 38040 9737 i
*> 1.0.128.0/17     213.200.64.93            0             0 3257 38040 9737 9737 i
*> 1.0.129.0/24     213.200.64.93            0             0 3257 4651 9737 9737 23969 i
*> 1.0.130.0/24     213.200.64.93            0             0 3257 6453 4651 9737 9737 9737 23969 i
*> 1.0.131.0/24     213.200.64.93            0             0 3257 6453 4651 9737 9737 9737 23969 i
*> 1.0.142.0/23     213.200.64.93            0             0 3257 6453 4651 9737 9737 9737 23969 i
*> 1.0.160.0/19     213.200.64.93           18             0 3257 2914 38040 9737 i
*> 1.0.192.0/21     213.200.64.93            0             0 3257 6453 4651 9737 9737 9737 23969 i
</code></pre>

Plenty of prefixes to play with…let’s try a couple of examples now shall we?

## Only allow prefixes that originated from AS 3257

This example will only accept prefixes that originated in AS 3257, all the other prefixes won’t be permitted:

<pre><code><strong>route-server.as3257.net>show ip bgp regexp ^3257$
</strong>BGP table version is 4492538, local router ID is 213.200.87.253
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 2.16.0.0/23      213.200.64.93          154             0 3257 i
*> 2.16.4.0/24      213.200.64.93          230             0 3257 i
*> 2.16.5.0/24      213.200.64.93          230             0 3257 i
*> 2.16.34.0/24     213.200.64.93           80             0 3257 i
</code></pre>

Let me explain the regular expression that I used here. The ^ symbol means that this is the beginning of the string and the $ matches the end of the string. We put 3257 in between so only “3257” matches. If you want to configure this filter on a Cisco IOS router you can do this with the **as-path access-list** command:

```
ip as-path access-list 1 permit ^3257$

route-map AS_PATH_FILTER permit 10
match as-path 1

router bgp 1
neighbor 213.200.64.93 remote-as 3257
neighbor 213.200.64.93 route-map AS_PATH_FILTER in
```

The as-path access-list works like the normal access-lists, there is a hidden “deny any” at the bottom. First we create the as-path access-list and then attach it to a route-map. In the BGP configuration you can attach the route-map to one of your BGP neighbors.

Let’s look at another example…

## Only allow networks that passed through AS 3257

We only want to see prefixes that passed through AS 3257, here’s how:

<pre><code><strong>route-server.as3257.net>show ip bgp regexp _3257_
</strong>BGP table version is 4492787, local router ID is 213.200.87.253
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.0.0.0/24       213.200.64.93            0             0 3257 15169 i
*> 1.0.4.0/24       213.200.64.93            0             0 3257 6453 7545 56203 i
*> 1.0.5.0/24       213.200.64.93            0             0 3257 6453 7545 56203 i
*> 1.0.6.0/24       213.200.64.93            0             0 3257 174 4826 38803 56203 i
*> 1.0.7.0/24       213.200.64.93            0             0 3257 174 4826 38803 56203 i
*> 1.0.20.0/23      213.200.64.93         1551             0 3257 2516 2519 i
</code></pre>

The regular expression starts and ends with a \_ . This matches the space between the AS path numbers. I’m not using a ^or $ to indicate the start and end of the string so there can be as many autonomous systems as we want, as long as it passed through AS 3257 it will match. Here’s what it looks like on a router:

```
ip as-path access-list 1 permit _3257_

route-map AS_PATH_FILTER permit 10
match as-path 1

router bgp 1
neighbor 213.200.64.93 remote-as 3257
neighbor 213.200.64.93 route-map AS_PATH_FILTER in
```

I got a few more examples…

## Deny prefixes that originated from AS 56203 and permit everything else

This one might be useful if you want to block prefixes that originated in a particular AS:

<pre><code><strong>route-server.as3257.net>show ip bgp regexp _56203$
</strong>BGP table version is 4493689, local router ID is 213.200.87.253
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.0.4.0/24       213.200.64.93            0             0 3257 6453 7545 56203 i
*> 1.0.5.0/24       213.200.64.93            0             0 3257 6453 7545 56203 i
*> 1.0.6.0/24       213.200.64.93            0             0 3257 174 4826 38803 56203 i
*> 1.0.7.0/24       213.200.64.93            0             0 3257 174 4826 38803 56203 i
*> 103.2.178.0/24   213.200.64.93            0             0 3257 174 4826 38803 56203 i
*> 103.2.179.0/24   213.200.64.93            0             0 3257 174 4826 38803 56203 i
</code></pre>

The first AS is always on the right side, so in order to match this we end the string with a $ and put the AS number just in front of it. The \_ will match the space in front of the AS number. On a router it will look like this:

```
ip as-path access-list 1 deny _56203$
ip as-path access-list 1 permit .*

route-map AS_PATH_FILTER permit 10
match as-path 1

router bgp 1
neighbor 213.200.64.93 remote-as 3257
neighbor 213.200.64.93 route-map AS_PATH_FILTER in
```

First we use a deny statement to block the AS number and then we use a permit .\* to allow everything else.  The . (dot) matches anything and the \* (wildcard) means “repeat the previous character zero or many times”. This will permit everything.

The next one is a bit more complicated…

## Allow prefixes from AS 3257 and its directly connected ASes but deny the rest

This one lets us accept all prefixes from AS 3257 and the directly connected autonomous systems of AS 3257:

<pre><code><strong>route-server.as3257.net>show ip bgp regexp ^3257_[0-9]*$
</strong>BGP table version is 4493802, local router ID is 213.200.87.253
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 1.0.0.0/24       213.200.64.93            0             0 3257 15169 i
*> 1.1.1.0/24       213.200.64.93            0             0 3257 15169 i
*> 1.2.3.0/24       213.200.64.93            0             0 3257 15169 i
*> 1.9.0.0/16       213.200.64.93          135             0 3257 4788 i
*> 1.9.52.0/24      213.200.64.93          170             0 3257 4788 ?
</code></pre>

We start with the ^3257 so that we only accept prefixes from AS 3257. The \_ will match on the space and the \[0-9] will match on any character between 0 and 9. The \* means that we repeat the last character (0-9). This means that AS1 would match, but also AS123 or AS12345, etc. The $ at the end will make sure that only 1 autonomous system behind AS 3257 is allowed.

Here’s what this one looks like on a router:

```
ip as-path access-list 1 permit ^3257_[0-9]*$

route-map AS_PATH_FILTER permit 10
match as-path 1

router bgp 1
neighbor 213.200.64.93 remote-as 3257
neighbor 213.200.64.93 route-map AS_PATH_FILTER in
```

That’s it for now! I hope these examples are helpful to understand regular expressions a bit more and how to configure the as-path access-list on a Cisco IOS router. If you have any questions, feel free to ask.
