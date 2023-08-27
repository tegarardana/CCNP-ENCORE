# How to read the BGP Table

All prefixes that BGP learns are stored in the BGP table. In this lesson we’ll take a look at this table and you will learn how to read it. We’ll start with a simple topology and finish with a quick peek at a full Internet routing table.

## Configuration

Here’s the topology we will use. 4 routers, each in a different autonomous system:\


<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-as1-as2-as3-as41.png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
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
interface fastEthernet0/1
 ip address 192.168.24.2 255.255.255.0
!
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 neighbor 192.168.24.4 remote-as 4
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
interface fastEthernet0/0
 ip address 192.168.13.3 255.255.255.0
!
interface fastEthernet0/1
 ip address 192.168.34.3 255.255.255.0
!
router bgp 3
 neighbor 192.168.13.1 remote-as 1
 neighbor 192.168.34.4 remote-as 4
!
end
```
{% endtab %}

{% tab title="R4" %}
```
hostname R4
!
interface Loopback 0
 ip address 4.4.4.4 255.255.255.255
!
interface fastEthernet0/0
 ip address 192.168.24.4 255.255.255.0
!
interface fastEthernet0/1
 ip address 192.168.34.4 255.255.255.0
!
router bgp 4
 network 4.4.4.4 mask 255.255.255.255
 neighbor 192.168.24.2 remote-as 2
 neighbor 192.168.34.3 remote-as 3
!
end
```
{% endtab %}
{% endtabs %}

The BGP configurations are pretty straight-forward, we are using eBGP here. Note that R4 advertises a network (loopback interface) in BGP.

## Reading the BGP Table

Let’s take a look at the BGP tables. We’ll start with R4:

<pre><code><strong>R4#show ip bgp
</strong>BGP table version is 2, local router ID is 192.168.34.4
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 4.4.4.4/32       0.0.0.0                  0         32768 i
</code></pre>

Ok so what do we see here? Let’s start with the items I highlighted in red first. This router has network 4.4.4.4/32 in its BGP table and in front of the network there’s the **\*>** symbol:

* The **\*** means that this is a valid route and that BGP is able to use it.
* The **>** means that this entry has been selected as the best path.

The **next hop** is 0.0.0.0. The next hop of 0.0.0.0 means that this network originated on this router, that makes sense since I used the network command on R4 to advertise this network into BGP.

Further to the right you see [metric](https://networklessons.com/cisco/ccnp-encor-350-401/how-to-configure-bgp-med-attribute), [local preference](https://networklessons.com/cisco/ccnp-encor-350-401/how-to-configure-bgp-local-preference-attribute) and [weight](https://networklessons.com/cisco/ccnp-encor-350-401/how-to-configure-bgp-weight-attribute). These are the BGP attributes that are used to select the best path.

**Path** will show the AS path, there’s nothing there since this network was advertised in BGP on this router. On the other routers you’ll see something here.

The **‘i’** is the **origin code** and indicates that this network was advertised into BGP using the network command, the table says it refers to IGP but it doesn’t have anything to do with “interior gateway protocols”. When you redistribute something into BGP it will show up with the **?** symbol. You will never see the **‘e’** symbol, this refers to EGP (Exterior Gateway Protocol) which is the predecessor of BGP.

Some of the other things you see here is the BGP table version, every time the best path changes this number will increase. You can see the BGP router ID of this router and there are some other status codes:

* **supressed**: BGP knows the network but won’t advertise it, this can occur when the network is part of a summary.
* **damped**: BGP doesn’t advertise this network because it was flapping too often (network appears, disapears, appears, etc.) so it got a penalty.
* **history**: BGP learned this network but doesn’t have a valid route at the moment.
* **RIB-failure**: BGP learned this network but didn’t install it in the routing table. This occurs when another routing protocol with a lower administrative distance also learned it.
* **stale**: this is used for non-stop forwarding, this entry has to be refreshed when the remote BGP neighbor has returned.

Let’s look at the BGP tables of the other routers, we’ll continue with R2:

<pre><code><strong>R2#show ip bgp
</strong>BGP table version is 2, local router ID is 192.168.24.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 4.4.4.4/32       192.168.24.4             0             0 4 i
</code></pre>

The output of R2 is similar to what we have seen on R4 but there are two important differences. The first one is the next hop, R2 learned about this network from 192.168.24.4. The second thing is the AS path, it’s showing AS 4.

Let’s check R1:

<pre><code><strong>R1#show ip bgp
</strong>BGP table version is 2, local router ID is 192.168.13.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  4.4.4.4/32       192.168.13.3                           0 3 4 i
*>                  192.168.12.2                           0 2 4 i
</code></pre>

This output is even more interesting, this router has learned about our network from R2 and R3. Both entries are valid, they have \* in front of them. BGP selected the path through R2 as the best path, you can see the > in front of this entry.You can also see the AS paths to reach this network.

The best path can be found in the routing table:

<pre><code><strong>R1#show ip route bgp
</strong>     4.0.0.0/32 is subnetted, 1 subnets
B       4.4.4.4 [20/0] via 192.168.12.2, 00:25:51
</code></pre>

If you want you can take a closer look at one of the entries in the BGP table, this is useful when you have a lot of networks:

<pre><code><strong>R1#show ip bgp 4.4.4.4
</strong>BGP routing table entry for 4.4.4.4/32, version 2
Paths: (2 available, best #2, table Default-IP-Routing-Table)
  Advertised to update-groups:
        1
  3 4
    192.168.13.3 from 192.168.13.3 (192.168.34.3)
      Origin IGP, localpref 100, valid, external
  2 4
    192.168.12.2 from 192.168.12.2 (192.168.24.2)
      Origin IGP, localpref 100, valid, external, best
</code></pre>

The information you see above tells us that we have two paths for this network, the second one has been selected as the best path. Last but not least, let’s take a look at R3:

<pre><code><strong>R3#show ip bgp
</strong>BGP table version is 2, local router ID is 192.168.34.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  4.4.4.4/32       192.168.13.1                           0 1 2 4 i
*>                  192.168.34.4             0             0 4 i
</code></pre>

Above you can see that R3 has two entries, it can use R1 or R4 to reach 4.4.4.4/32.

## BGP Looking Glass Server

You now have an idea how to read the BGP table, let’s take a look at an actual BGP table that is used on the Internet. We can use a [looking glass server](http://www.bgp4.as/looking-glasses) for this. These routers provide read-only access so you can take a look at their BGP tables. I’ll use the route-views.optus.net.au server for this.

<pre><code><strong>route-views.optus.net.au>show ip bgp summary
</strong>BGP router identifier 203.202.125.6, local AS number 65535
BGP table version is 244799119, main routing table version 244799119
539211 network entries using 69019008 bytes of memory
2321994 path entries using 120743688 bytes of memory
200268/89271 BGP path/bestpath attribute entries using 24833232 bytes of memory
78014 BGP AS-PATH entries using 3503222 bytes of memory
3793 BGP community entries using 321910 bytes of memory
18 BGP extended community entries using 448 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 218421508 total bytes of memory
Dampening enabled. 557 history paths, 440 dampened paths
BGP activity 3726941/3187730 prefixes, 16191208/13869214 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.65.89.161   4         7474 1318568  910036 244799121    0    0 41w2d       21006
202.139.124.130 4         7474 1306375  910020 244799121    0    0 41w2d       21024
202.160.242.71  4         7473       0       0        1    0    0 never    Active
203.13.132.29   4         7474 2107424  910019 244799121    0    0 41w2d       20649
203.13.132.35   4         7474 59948648  910162 244799121    0    0 41w2d      538696
203.13.132.37   4         7474 1571073  601855 244799121    0    0 27w2d       20651
203.13.132.41   4         7474 2254818  910043 244799121    0    0 41w2d       20649
203.13.132.47   4         7474   46416   19778 244799121    0    0 6d07h       20649
203.13.132.49   4         7474 2260238  910030 244799121    0    0 41w2d       20649
203.13.132.51   4         7474 2296993  910146 244799121    0    0 41w2d       20649
203.13.132.53   4         7474 59909540  910088 244799121    0    0 41w2d      538696
203.202.143.3   4         7474       0       0        1    0    0 never    Idle (Admin)
203.202.143.33  4         7474 34662511  910049 244799121    0    0 41w2d      539054
203.202.143.34  4         7474 33523616  910040 244799121    0    0 41w2d      539065
</code></pre>

This router has over 500.000 networks and knows about more than 2.000.000 paths for these networks. It is connected to 14 neighbors (2 are down) and here’s what the BGP table looks like:

<pre><code><strong>route-views.optus.net.au>show ip bgp
</strong>BGP table version is 244797821, local router ID is 203.202.125.6
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  1.0.0.0/24       203.13.132.47                          0 7474 15169 i
*                   203.13.132.37                          0 7474 15169 i
*                   192.65.89.161            1             0 7474 15169 i
*                   203.202.143.34                         0 7474 15169 i
*>                  203.202.143.33                         0 7474 15169 i
*                   203.13.132.49                          0 7474 15169 i
*                   202.139.124.130          1             0 7474 15169 i
*                   203.13.132.51                          0 7474 15169 i
*                   203.13.132.53                          0 7474 15169 i
*                   203.13.132.41                          0 7474 15169 i
*                   203.13.132.35                          0 7474 15169 i
*                   203.13.132.29                          0 7474 15169 i
*  1.0.4.0/24       203.13.132.47           10             0 7474 4826 56203 i
*                   203.13.132.37           10             0 7474 4826 56203 i
*                   203.202.143.34                         0 7474 4826 56203 i
*>                  203.202.143.33                         0 7474 4826 56203 i
*                   203.13.132.49           10             0 7474 4826 56203 i
*                   192.65.89.161            1             0 7474 4826 56203 i
*                   203.13.132.51            1             0 7474 4826 56203 i
*                   202.139.124.130          1             0 7474 4826 56203 i
*                   203.13.132.53            1             0 7474 4826 56203 i
*                   203.13.132.41            1             0 7474 4826 56203 i
*                   203.13.132.35            1             0 7474 4826 56203 i
*                   203.13.132.29            1             0 7474 4826 56203 i
</code></pre>

You can keep pressing enter, this BGP table is very long. I’m only showing the first two networks. As you can see this router has network 1.0.0.0/24 in its BGP table and knows about 12 different paths to get there. It decided to use 203.202.143.33 as the next hop.

It also learned about network 1.0.4.0/24 and is using 203.202.143.33 as the next hop.

This router probably knows about a couple of networks with issues, a fun way to find these is by searching in the BGP table and excluding everything that starts with a \*. This removes all the valid networks from the BGP table:

<pre><code><strong>route-views.optus.net.au>show ip bgp | exclude *
</strong>BGP table version is 244799956, local router ID is 203.202.125.6
              r RIB-failure, S Stale, m multipath, b backup-path, x best-external
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
 d 2.93.235.0/24    203.202.143.33                         0 7474 7473 3549 701 703 702 3216 8402 ?
 d                  203.202.143.34                         0 7474 7473 3549 701 703 702 3216 8402 ?
 d                  203.13.132.35            1             0 7474 7473 3549 701 703 702 3216 8402 ?
 d                  203.13.132.53            1             0 7474 7473 3549 701 703 702 3216 8402 ?
 h                  203.13.132.53            1             0 7474 7473 1299 6453 4755 45820 45117 i
 h                  203.13.132.35            1             0 7474 7473 1299 6453 4755 45820 45117 i
 h 31.131.7.0/24    203.202.143.33                         0 7474 7473 1299 46786 5577 i
 h                  203.202.143.34                         0 7474 7473 1299 46786 5577 i
 h                  203.13.132.53            1             0 7474 7473 1299 46786 5577 i
 h                  203.13.132.35            1             0 7474 7473 1299 46786 5577 i
</code></pre>

I’m using the exclude command to filter every line that has a \* in it. I have to use the symbol in front of it since the \* is a wildcard for regular expressions.

Above you can see two networks with issues. The first one (2.93.235.0/24) has a **d** in front of it which means it’s dampened. The second network 31.131.7.0/24 has a **h** in front of it that indicates that it’s a history entry.

That’s all I have for now about the BGP table, I hope this has been useful to understand how to read it. It might be a fun idea to play with one of the looking glass servers yourself!

If you have any questions, just leave a comment.
