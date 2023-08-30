# IGMP Version 1

IGMP (Internet Group Management Protocol) version 1 is the first version that hosts can use to announce to a router that they want to receive multicast traffic from a specific group. It’s a simple protocol that uses only two messages:

* Membership report
* Membership query

When a host wants to join a multicast group, it will send a **membership report** to the group address that it wants to receive. When the multicast-enabled router receives this message, it will start forwarding the requested multicast traffic on the interface where it received the IGMP membership report on.

The router will periodically send a **membership query** to destination 224.0.0.1 (all hosts multicast group address). Hosts that receive this message will respond with a membership report to tell the router that they are still interested in receiving the multicast traffic. When the router receives the membership report, it’s expiry timer will be refreshed. When no hosts respond, the router knows that nobody is interested anymore in the multicast traffic and it will then remove the entry once the timer exceeds.

In this lesson, we’ll take a closer look at these two messages and I will show you how to configure IGMP version 1. Here’s the topology we will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-topology-router-two-hosts.png" alt=""><figcaption></figcaption></figure>

Above we have one router and two hosts. On the router, we will enable multicast routing and IGMP on its Gigabit 0/1 interface.

All modern operating systems support IGMP, you can use Windows, Linux or MacOS. You can also use Cisco routers as host devices which is what I will do in this example.

Let’s configure R1 first:

<pre><code><strong>R1(config)#ip multicast-routing
</strong></code></pre>

<pre><code><strong>R1(config)#interface GigabitEthernet 0/1
</strong><strong>R1(config-if)#ip igmp version 1
</strong><strong>R1(config-if)#ip pim sparse-mode 
</strong></code></pre>

We configured three commands on the router:

* First we enabled multicast routing globally, this is required for the router to process IGMP traffic.
* We enabled PIM on the interface. PIM is used for multicast routing between routers and is also required for the router to process IGMP traffic.
* We changed IGMP to version 1, the default on Cisco IOS routers is version 2.

Let’s verify if IGMP is enabled on our router:

<pre><code><strong>R1#show ip igmp interface GigabitEthernet 0/1
</strong>GigabitEthernet0/1 is up, line protocol is up
  Internet address is 192.168.1.1/24
  IGMP is enabled on interface
  Current IGMP host version is 1
  Current IGMP router version is 1
  IGMP query interval is 60 seconds
  IGMP configured query interval is 60 seconds
  Inbound IGMP access group is not set
  IGMP activity: 1 joins, 0 leaves
  Multicast routing is enabled on interface
  Multicast TTL threshold is 0
  Multicast designated router (DR) is 192.168.1.1 (this system)
  IGMP querying router is 192.168.1.1 (this system)
  Multicast groups joined by this system (number of users):
      224.0.1.40(1)
</code></pre>

Above we can see that IGMP is enabled on the interface. Each 60 seconds it will send a membership query on this interface. Here’s what that looks like in wireshark:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/11/wireshark-capture-multicast-igmp-version-1-membership-query.png" alt=""><figcaption></figcaption></figure>

Above you can see the membership query packet that our router sends to 224.0.0.1. There’s not much in the packet but when the hosts receive this, they will act upon it. Let’s configure one of our hosts so that it joins a multicast group:

<pre><code><strong>H1(config)#interface GigabitEthernet 0/1
</strong><strong>H1(config-if)#ip igmp version 1
</strong><strong>H1(config-if)#ip igmp join-group 239.1.1.1
</strong></code></pre>

Above we configured our host to join multicast group 239.1.1.1.1. It will now send a membership report:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/11/multicast-igmp-version-1-membership-report.png" alt=""><figcaption></figcaption></figure>

Above you can see that host 1 sends a membership report with destination 239.1.1.1.1. When our router receives this, it will forward traffic with destination 239.1.1.1 on its interface where it received this membership report.

We can also see these packets with a debug. Let’s enable IGMP debugging on our router and two hosts:

<pre><code>R1, H1 &#x26; H2
<strong>#debug ip igmp 
</strong>IGMP debugging is on
</code></pre>

Now I will configure our second host to join the same multicast group address:

<pre><code><strong>H2(config)#interface GigabitEthernet 0/1
</strong><strong>H2(config-if)#ip igmp version 1
</strong><strong>H2(config-if)#ip igmp join-group 239.1.1.1  
</strong></code></pre>

Here’s what you will see on the router:

```
R1#
IGMP(0): Send v1 general Query on GigabitEthernet0/1
IGMP(0): Set report delay time to 0.7 seconds for 224.0.1.40 on GigabitEthernet0/1
IGMP(0): Received v1 Report on GigabitEthernet0/1 from 192.168.1.102 for 239.1.1.1
IGMP(0): Received Group record for group 239.1.1.1, mode 2 from 192.168.1.102 for 0 sources
IGMP(0): Updating EXCLUDE group timer for 239.1.1.1
IGMP(0): MRT Add/Update GigabitEthernet0/1 for (*,239.1.1.1) by 0
IGMP(0): Received v1 Report on GigabitEthernet0/1 from 192.168.1.101 for 239.1.1.1
IGMP(0): Received Group record for group 239.1.1.1, mode 2 from 192.168.1.101 for 0 sources
IGMP(0): Updating EXCLUDE group timer for 239.1.1.1
```

Above you can see that the router sends a membership query and that it receives two membership reports, one from each host. Here’s what the debug on the hosts looks like:

```
H1#
IGMP(0): Received v1 Query on GigabitEthernet0/1 from 192.168.1.1
IGMP(0): Starting v1 router present timer on GigabitEthernet0/1
IGMP(0): Set report delay time to 0.8 seconds for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Received v1 Report on GigabitEthernet0/1 from 192.168.1.102 for 239.1.1.1
IGMP(0): Received Group record for group 239.1.1.1, mode 2 from 192.168.1.102 for 0 sources
IGMP(0): Send v1 Report for 239.1.1.1 on GigabitEthernet0/1
```

```
H2#
IGMP(0): Received v1 Query on GigabitEthernet0/1 from 192.168.1.1
IGMP(0): Starting v1 router present timer on GigabitEthernet0/1
IGMP(0): Set report delay time to 0.6 seconds for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Send v1 Report for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): Received v1 Report on GigabitEthernet0/1 from 192.168.1.101 for 239.1.1.1
IGMP(0): Received Group record for group 239.1.1.1, mode 2 from 192.168.1.101 for 0 sources
```

The hosts received the membership query and send membership reports in return. This will happen over and over again, you can see it in the following packet capture:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/11/wireshark-capture-multicast-igmp-membership-query-report.png" alt=""><figcaption></figcaption></figure>

Above you can see that each 60 seconds the router sends a membership query. I’m showing only one of the hosts which responds with a membership report.

Let’s see what happens when our hosts don’t want to receive multicast traffic anymore:

<pre><code>H1 &#x26; H2#
<strong>(config)#interface GigabitEthernet 0/1
</strong><strong>(config-if)#no ip igmp join-group 239.1.1.1
</strong></code></pre>

This is what you will see on the hosts:

```
H1 & H2#
IGMP(0): IGMP delete group 239.1.1.1 on GigabitEthernet0/1
```

Our hosts are no longer interested in multicast group address 239.1.1.1 but this is something that they **don’t report** to the router. The router will keep forwarding multicast traffic to the hosts until the timer expires. We can see the current timer below:

<pre><code><strong>R1#show ip igmp groups 239.1.1.1
</strong>IGMP Connected Group Membership
Group Address    Interface                Uptime    Expires   Last Reporter   Group Accounted
239.1.1.1        GigabitEthernet0/1       19:22:56  00:01:07  192.168.1.101
</code></pre>

The current time expires in one minute and seven seconds. When it expires, this is what you will see:

```
R1#
IGMP(0): Switching to INCLUDE mode for 239.1.1.1 on GigabitEthernet0/1
IGMP(0): MRT delete GigabitEthernet0/1 for (*,239.1.1.1) by 0
```

Our router didn’t receive any membership reports anymore from the hosts so the timer expired. As a result, it deletes multicast group 239.1.1.1 from the multicast routing table and it will be no longer forwarded.

Want to take a look at the wireshark captures yourself?

[IGMP version 1 membership report](https://www.cloudshark.org/captures/8c4e237823a6)

[IGMP version 1 membership query](https://www.cloudshark.org/captures/4a3f5f345e8b)

[IGMP version 1 membership query and report](https://www.cloudshark.org/captures/7bb04610bb7a)

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
 ip igmp version 1
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
 ip igmp version 1
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
 ip igmp version 1
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

IGMP version 1 is a simple protocol with only two different packets. One of the downsides of IGMP version 1 is that the router will keep forwarding multicast traffic even when there are no more hosts interested in our traffic. IGMP version 2 fixes this, we’ll take a look at this in the next lesson.
