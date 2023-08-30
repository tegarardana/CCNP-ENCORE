# IGMP Version 3

IGMP version 3 adds support for “source filtering”. IGMP [version 1](https://networklessons.com/cisco/ccnp-encor-350-401/igmp-version-1) and [version 2](https://networklessons.com/cisco/ccnp-encor-350-401/igmp-version-2) allow hosts to join multicast groups but they don’t check the source of the traffic. Any source is able to receive traffic to the multicast group(s) that they joined.

With source filtering, we can join multicast groups but only from **specified source addresses.** IGMP version 3 is a requirement for SSM (Source Specific Multicast) which we will cover in another lesson.

Why is this useful? Let me give you an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-video-server-four-hosts.png" alt=""><figcaption></figcaption></figure>

Above we have a video server that is streaming multicast traffic on the network using destination address 239.1.1.1. There are four hosts listening to this traffic, life is good. Suddenly something happens:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-attacker-sending-traffic.png" alt=""><figcaption></figcaption></figure>

An attacker didn’t like the video stream and decided to stream his [favorite video](https://www.youtube.com/watch?v=dQw4w9WgXcQ) to destination address 239.1.1.1.1. Since we don’t check the source address, everyone will receive the traffic from our attacker. It’s also possible to send bogus traffic and create a DoS attack like this.

IGMP version 1 and 2 don’t have any protection against this.

With IGMP version 3, our hosts can be configured to receive multicast traffic only from specified source addresses. Let’s see how this works, I’ll use the following topology for this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-3-topology.png" alt=""><figcaption></figcaption></figure>

We will only use two devices, one multicast enabled router and a host device. I’m using a Cisco router as the host device as well.

Let’s start with R1:

<pre><code><strong>R1(config)#ip multicast-routing
</strong><strong>R1(config)#interface GigabitEthernet 0/1
</strong><strong>R1(config-if)#ip pim sparse-mode 
</strong><strong>R1(config-if)#ip igmp version 3
</strong></code></pre>

Our router requires multicast routing and PIM should be enabled on the interface. The default version of IGMP is 2 so we’ll change it to version 3. Before we let H1 join a multicast group, let’s enable debugging on both devices:

<pre><code><strong>R1 &#x26; H1#debug ip igmp 
</strong>IGMP debugging is on
</code></pre>

R1 will start sending membership general queries like the one below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-3-membership-general-query.png" alt=""><figcaption></figcaption></figure>

Let’s configure H1 to join a multicast group:

<pre><code><strong>H1(config)#interface GigabitEthernet 0/1
</strong><strong>H1(config-if)#ip igmp join-group 239.1.1.1 ?
</strong>  source  Include SSM source
  &#x3C;cr>
</code></pre>

Besides configuring a group, I can configure the host to include a source address. Let’s pick something:

<pre><code><strong>H1(config-if)#ip igmp join-group source 239.1.1.1 1.1.1.1
</strong></code></pre>

H1 will now include the source address in its membership report messages. Here’s what you will see on the console:

```
H1#
IGMP(0): WAVL Insert group: 239.1.1.1 interface: GigabitEthernet0/1Successful
IGMP(0): Create source 1.1.1.1
IGMP(0): Building v3 Report on GigabitEthernet0/1
IGMP(0): Add Group Record for 239.1.1.1, type 5
IGMP(0): Add Source Record 1.1.1.1
IGMP(0): Add Group Record for 239.1.1.1, type 6
IGMP(0): No sources to add, group record removed from report
IGMP(0): Send unsolicited v3 Report with 1 group records on GigabitEthernet0/1
IGMP(0): Building v3 Report on GigabitEthernet0/1
IGMP(0): Add Group Record for 239.1.1.1, type 5
IGMP(0): Add Source Record 1.1.1.1
IGMP(0): Add Group Record for 239.1.1.1, type 6
IGMP(0): No sources to add, group record removed from report
IGMP(0): Send unsolicited v3 Report with 1 group records on GigabitEthernet0/1
```

H1 sends two membership report messages. The first message includes the multicast group address and source address that we want to receive. The second message includes the “mode”. There are two modes:

* Include: this is a list of source addresses that we **accept** multicast traffic from, everything else should not be forwarded.
* Exclude: this is a list of source addresses that we **refuse** to accept multicast traffic from, everything else should be forwarded.

Here’s what it looks like in wireshark:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-3-membership-report-new-source.png" alt=""><figcaption></figcaption></figure>

Above you can see that H1 wants to receive traffic from 239.1.1.1 from source 1.1.1.1. The destination for these membership reports is 224.0.0.22 which is the IGMP version 3 “all routers” group address. IGMP version 1 and 2 used the 224.0.0.2 (all routers) address.

The next membership report from H1 tells the router to use include mode:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-3-membership-report-for-source.png" alt=""><figcaption></figcaption></figure>

Here’s what the debug looks like on R1:

```
R1#
IGMP(0): Received v3 Report for 1 group on GigabitEthernet0/1 from 192.168.1.101
IGMP(0): Received Group record for group 239.1.1.1, mode 5 from 192.168.1.101 for 1 sources
IGMP(0): WAVL Insert group: 239.1.1.1 interface: GigabitEthernet0/1Successful
IGMP(0): Create source 1.1.1.1
IGMP(0): Updating expiration time on (1.1.1.1,239.1.1.1) to 180 secs
IGMP(0): Setting source flags 4 on (1.1.1.1,239.1.1.1)
IGMP(0): MRT Add/Update GigabitEthernet0/1 for (*,239.1.1.1) by 0
IGMP(0): Received v3 Report for 1 group on GigabitEthernet0/1 from 192.168.1.101
IGMP(0): Received Group record for group 239.1.1.1, mode 5 from 192.168.1.101 for 1 sources
IGMP(0): Updating expiration time on (1.1.1.1,239.1.1.1) to 180 secs
IGMP(0): MRT Add/Update GigabitEthernet0/1 for (*,239.1.1.1) by 0
```

R1 receives the membership reports from H1 and adds an entry in the multicast routing table. We can also verify this with the following command:

<pre><code><strong>R1#show ip igmp groups 239.1.1.1 detail 
</strong>
Flags: L - Local, U - User, SG - Static Group, VG - Virtual Group,
       SS - Static Source, VS - Virtual Source,
       Ac - Group accounted towards access control limit

Interface:      GigabitEthernet0/1
Group:          239.1.1.1
Flags:
Uptime:         00:12:27
Group mode:     INCLUDE
Last reporter:  192.168.1.101
Group source list: (C - Cisco Src Report, U - URD, R - Remote, S - Static,
                    V - Virtual, M - SSM Mapping, L - Local,
                    Ac - Channel accounted towards access control limit)
  Source Address   Uptime    v3 Exp   CSR Exp   Fwd  Flags
  1.1.1.1          00:12:27  00:02:27  stopped   Yes  R
</code></pre>

Above you can see that R1 keeps track of which multicast traffic it should forward and from which source addresses. The group mode is “include” which is what H1 requested.

What happens when H1 leaves a group? Let’s find out:

<pre><code><strong>H1(config)#interface Gigabit 0/1  
</strong><strong>H1(config-if)#no ip igmp join-group 239.1.1.1 source 1.1.1.1
</strong></code></pre>

H1 will send a membership report to indicate that it doesn’t want to receive the multicast traffic from this source anymore:

![Multicast IGMP Version 3 membership report block old sources](https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-3-membership-report-block-old-sources.png)

Above you can see that the membership report has been sent to destination 224.0.0.22. The record type is “block old sources” and includes the multicast group address and source address that we no longer want to receive.

When R1 receives this packet, it will send a membership query for this specific multicast address and source address:

![Multicast IGMP Version 3 membership specific query](https://cdn.networklessons.com/wp-content/uploads/2014/08/multicast-igmp-version-3-membership-specific-query.png)

Above you can see the membership query from R1 which is specific for multicast group address 239.1.1.1 using source 1.1.1.1.

Want to take a look at these packet captures yourself?

[Multicast IGMP version 3 membership query general](https://www.cloudshark.org/captures/9e88efd7687f)

[Multicast IGMP version 3 membership report](https://www.cloudshark.org/captures/296ec7dbf3f6)

[Multicast IGMP version 3 leave group](https://www.cloudshark.org/captures/a7b0fa61d5af)

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
 ip igmp version 3
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
 ip igmp version 3 
 ip igmp join-group 239.1.1.1 source 1.1.1.1
 duplex auto
 speed auto
 media-type rj45
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

You have now seen the basics of how IGMP version 3 operates. Unlike version 1/2, we are able to join a multicast group address with a specified source address. In another lesson where we look at SSM (Source Specific Multicast) we will dive deeper into IGMP version 3.

I hope you enjoyed this lesson, if you have any questions feel free to leave a comment.
