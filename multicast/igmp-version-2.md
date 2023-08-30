# IGMP Version 2

IGMP version 2 is the “enhanced” version of [IGMP version 1](https://networklessons.com/cisco/ccnp-encor-350-401/igmp-version-1). One of the major reasons for a new version was to improve the “leave” mechanism. In IGMP version 1, hosts just stop listening to the multicast group address but they never report this to the router. Here are the new features:

* **Leave group messages**: when a host no longer wants to listen to a multicast group address then it will report to the router that it has stopped listening.
* **Group specific membership query**: the router is now able to send a membership query for a specific group address. When the router receives a leave group message, it will use this query to check if there are still any hosts interested in receiving the multicast traffic.
* **MRT (Maximum Response Time) field**: this is a new field in query messages. It specifies how much time hosts have to respond to the query. I will explain later why we use this with an example.
* **Querier election process**: when there are two routers in the same subnet then only one of them should send query messages. The election ensures only one router becomes the active querier. The router with the lowest IP address becomes the active querier.

To demonstrate these new features, I’ll use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-topology-router-two-hosts.png" alt=""><figcaption></figcaption></figure>

Above we have one multicast-enabled router and two hosts.

Let’s start with R1:

<pre><code><strong>R1(config)#ip multicast-routing
</strong><strong>R1(config)#interface GigabitEthernet 0/1
</strong><strong>R1(config-if)#ip pim sparse-mode 
</strong></code></pre>

First we have to enable multicast routing and PIM on the interface otherwise the router won’t process IGMP traffic. We can verify that it’s running:

<pre><code><strong>R1#show ip igmp interface GigabitEthernet 0/1
</strong>GigabitEthernet0/1 is up, line protocol is up
  Internet address is 192.168.1.1/24
  IGMP is enabled on interface
  Current IGMP host version is 2
  Current IGMP router version is 2
  IGMP query interval is 60 seconds
  IGMP configured query interval is 60 seconds
  IGMP querier timeout is 120 seconds
  IGMP configured querier timeout is 120 seconds
  IGMP max query response time is 10 seconds
  Last member query count is 2
  Last member query response interval is 1000 ms
  Inbound IGMP access group is not set
  IGMP activity: 6 joins, 5 leaves
  Multicast routing is enabled on interface
  Multicast TTL threshold is 0
  Multicast designated router (DR) is 192.168.1.1 (this system)
  IGMP querying router is 192.168.1.1 (this system)
  Multicast groups joined by this system (number of users):
      224.0.1.40(1)
</code></pre>

Above we can see that IGMP is enabled and that our router is the quering router. There are no other routers so that’s an easy way to win the election.

Before we let the hosts join a multicast group, let’s enable debugging on all devices:

<pre><code>R1, H1 &#x26; H2
<strong>#debug ip igmp 
</strong>IGMP debugging is on
</code></pre>

{% hint style="warning" %}
If you use a switch with [IGMP snooping](https://networklessons.com/cisco/ccnp-encor-350-401/igmp-snooping) enabled to connect the hosts and router, disable it, or you won’t see all packets.
{% endhint %}

On R1 you will see the following message:

```
R1#
IGMP(0): Send v2 general Query on GigabitEthernet0/1
```

Just like IGMP version 1, the router is now sending general membership queries every 60 seconds. This is what it looks like in wireshark:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-2-membership-query-general.png" alt=""><figcaption></figcaption></figure>

Above you can see the destination which is 224.0.0.1 (all hosts multicast group address). Let’s configure our first host to join a multicast group:

<pre><code><strong>H1(config)#interface GigabitEthernet 0/1
</strong><strong>H1(config-if)#ip igmp join-group 239.1.1.1
</strong></code></pre>

This is what you will see on the console of the host:

```
H1#
IGMP(0): WAVL Insert group: 239.1.1.1 interface: GigabitEthernet0/1Successful
IGMP(0): Send v2 Report for 239.1.1.1 on GigabitEthernet0/1
```

Our host is sending a membership report to 239.1.1.1 to tell the router that it wants to receive this multicast traffic. Here you can see the packet in wireshark:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-2-membership-report.png" alt=""><figcaption></figcaption></figure>

Here’s what R1 thinks of this:

```
R1#
IGMP(0): Received v2 Report on GigabitEthernet0/1 from 192.168.1.101 for 239.1.1.1
IGMP(0): Received Group record for group 239.1.1.1, mode 2 from 192.168.1.101 for 0 sources
IGMP(0): WAVL Insert group: 239.1.1.1 interface: GigabitEthernet0/1Successful
IGMP(0): Switching to EXCLUDE mode for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Updating EXCLUDE group timer for 239.1.1.1
IGMP(0): MRT Add/Update GigabitEthernet0/1 for (*,239.1.1.1) by 0
```

R1 receives the membership report from host 1 and adds an entry for multicast group 239.1.1.1.

Everything you have seen so far is pretty much the same as IGMP version 1. Our router sends general membership queries and the host sends a report.

When we add a second host, things will change. Let’s configure H2 to join the same group:

<pre><code><strong>H2(config)#interface GigabitEthernet 0/1
</strong><strong>H2(config-if)#ip igmp join-group 239.1.1.1
</strong></code></pre>

Here’s what we will see on the router:

```
R1#
IGMP(0): Send v2 general Query on GigabitEthernet0/1
IGMP(0): Received v2 Report on GigabitEthernet0/1 from 192.168.1.102 for 239.1.1.1
IGMP(0): Received Group record for group 239.1.1.1, mode 2 from 192.168.1.102 for 0 sources
IGMP(0): Updating EXCLUDE group timer for 239.1.1.1
IGMP(0): MRT Add/Update GigabitEthernet0/1 for (*,239.1.1.1) by 0
```

Our router sends another membership query and it also received the membership report from H2. Here’s what happens now when both hosts receive the query:

```
H1#
IGMP(0): Received v2 Query on GigabitEthernet0/1 from 192.168.1.1
IGMP(0): Set report delay time to 2.8 seconds for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Send v2 Report for 239.1.1.1 on GigabitEthernet0/1
```

```
H2#
IGMP(0): Received v2 Query on GigabitEthernet0/1 from 192.168.1.1
IGMP(0): Set report delay time to 3.0 seconds for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Received v2 Report on GigabitEthernet0/1 from 192.168.1.101 for 239.1.1.1
IGMP(0): Received Group record for group 239.1.1.1, mode 2 from 192.168.1.101 for 0 sources
IGMP(0): Cancel report for 239.1.1.1 on GigabitEthernet0/1
```

Above you can see that both hosts receive the query from the router. Host 1 sets its report delay time to 2.8 seconds and then sends the membership report.

H2 also receives the query but sets its report delay time to 3.0 seconds. Since it has to wait longer, H1 was able to send the membership report first. When H2 receives the report from H1 then it will cancel sending a membership report itself.

Where did these timers come from? This is one of the new features of IGMP version 2. The router advertises a maximum response time in its queries:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-2-maximum-response-time.png" alt=""><figcaption></figcaption></figure>

As the name implies, the maximum response time is the maximum amount of time our hosts have to respond to a membership query with a membership report. Each host picks a random time between 0 and the maximum response time (10 seconds in our example). When the timer expires, the host will send the membership report but **only if it doesn’t hear a report from another host**. This is called **report suppression.**

In our example, H1 picked a lower random number so it was able to send the membership report first. H2 noticed the report from H1 and didn’t send one anymore.

This is a great way to get rid of redundant reports. With 100 hosts on the network, with IGMP version 1 each of these would send a report to the router. With IGMP version 2, only one report is sent to the router.

The leave mechanism has also been improved. To demonstrate this, we’ll configure H2 to leave the group:

<pre><code><strong>H2(config)#interface GigabitEthernet 0/1
</strong><strong>H2(config-if)#no ip igmp join-group 239.1.1.1
</strong></code></pre>

Here’s what you will see on H2:

```
H2#
IGMP(0): IGMP delete group 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Send Leave for 239.1.1.1 on GigabitEthernet0/1
```

H2 is now sending a leave message. Here’s what happens on R1:

```
R1#
IGMP(0): Received Leave from 192.168.1.102 (GigabitEthernet0/1) for 239.1.1.1
IGMP(0): Received Group record for group 239.1.1.1, mode 3 from 192.168.1.102 for 0 sources
IGMP(0): Lower expiration timer to 2000 msec for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Send v2 Query on GigabitEthernet0/1 for group 239.1.1.1
IGMP(0): Received v2 Report on GigabitEthernet0/1 from 192.168.1.101 for 239.1.1.1
IGMP(0): Received Group record for group 239.1.1.1, mode 2 from 192.168.1.101 for 0 sources
IGMP(0): Updating EXCLUDE group timer for 239.1.1.1
IGMP(0): MRT Add/Update GigabitEthernet0/1 for (*,239.1.1.1) by 0
```

Above you can see that R1 receives the leave from H2. In response, it will send a membership query with destination 239.1.1.1 to figure out if there is still anyone else interested in receiving this traffic. After a while, it receives the membership report from H1 who is still interested.

Here’s what it looks like on H1:

```
H1#
IGMP(0): Received v2 Query on GigabitEthernet0/1 from 192.168.1.1
IGMP(0): Set report delay time to 1.0 seconds for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Received v2 Query on GigabitEthernet0/1 from 192.168.1.1
IGMP(0): Send v2 Report for 239.1.1.1 on GigabitEthernet0/1
```

Above you can see that H1 receives the membership query and responds with the membership report. Let’s see what this looks like in wireshark, first we have the leave message from H2:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-2-group-leave.png" alt=""><figcaption></figcaption></figure>

Above we see the leave group membership that H2 sends to destination 224.0.0.2 (all routers multicast group address). R1 will then send a membership query:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-2-membership-query.png" alt=""><figcaption></figcaption></figure>

Above you can see that this membership query was sent to 239.1.1.1. Last but not least, here’s the membership report from H1:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-2-membership-report.png" alt=""><figcaption></figcaption></figure>

Above you can see that H1 is still interested in receiving this multicast traffic since it sends a membership report with destination 239.1.1.1.

What happens when H1 also leaves the group? Let’s find out:

<pre><code><strong>H1(config)#interface GigabitEthernet 0/1
</strong><strong>H1(config-if)#no ip igmp join-group 239.1.1.1
</strong></code></pre>

Once we leave the group, H1 will send a leave immediately:

```
H1#
IGMP(0): IGMP delete group 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Send Leave for 239.1.1.1 on GigabitEthernet0/1
```

Here’s what the router does:

```
R1#
IGMP(0): Received Leave from 192.168.1.101 (GigabitEthernet0/1) for 239.1.1.1
IGMP(0): Received Group record for group 239.1.1.1, mode 3 from 192.168.1.101 for 0 sources
IGMP(0): Lower expiration timer to 2000 msec for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Send v2 Query on GigabitEthernet0/1 for group 239.1.1.1
IGMP(0): Switching to INCLUDE mode for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): MRT delete GigabitEthernet0/1 for (*,239.1.1.1) by 0
```

R1 receives the leave and sends a membership query. This time however there’s nobody interested anymore so the router deletes the group from the multicast routing table. We can also verify this with the following command:

<pre><code><strong>R1#show ip igmp groups 239.1.1.1
</strong>IGMP Connected Group Membership
Group Address    Interface                Uptime    Expires   Last Reporter   Group Accounted
</code></pre>

As you can see, we don’t have anyone that is interested in this group anymore.

Do you want to take a look at the wireshark captures yourself?

[Multicast IGMP Version 2 Query General](https://www.cloudshark.org/captures/611ec0d50369)

[Multicast IGMP Version 2 Report](https://www.cloudshark.org/captures/909766aa3ea6)

[Multicast IGMP Version 2 Leave Group](https://www.cloudshark.org/captures/7d789d000734)\


{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip multicast-routing 
!        
interface GigabitEthernet0/1
 ip address 192.168.1.1 255.255.255.0
 ip pim dense-mode
 ip igmp version 2
 duplex auto
 speed auto
 media-type rj45
!
end
```
{% endtab %}

{% tab title="H1" %}
```
hostname H1
!
interface GigabitEthernet0/1
 ip address 192.168.1.101 255.255.255.0
 ip igmp join-group 239.1.1.1
 duplex auto
 speed auto
 media-type rj45
!
end
```
{% endtab %}

{% tab title="H2" %}
```
hostname H2
!
interface GigabitEthernet0/1
 ip address 192.168.1.102 255.255.255.0
 ip igmp join-group 239.1.1.1
 duplex auto
 speed auto
 media-type rj45
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

IGMP version 2 is very similar to version 1 but because of the new features, it’s more efficient and leaving groups has become much faster. I hope you enjoyed this lesson, if you have any questions feel free to leave a comment.
