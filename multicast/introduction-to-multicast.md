# Introduction to Multicast

In this lesson we’ll take a look at the basics of multicast. You will learn what multicast is, how it works and the different protocols that are required.

First of all, let’s talk about what multicast is…

There are three types of traffic that we can choose from for our networks:

* Unicast
* Broadcast
* Multicast

If you want to send a message from **one** **source** to **one** **destination**, we use unicast. If you want to send a message from **one source** to **everyone**, we use broadcast.

What if we want to send a message from **one source to a group of receivers**? That’s when we use _multicast._&#x20;

## Unicast vs Broadcast vs Multicast

Why do you want to use multicast instead of unicast or broadcast? That’s best explained with an example. Let’s imagine that we want to stream a high definition video on the network using unicast, broadcast or multicast. You will see the advantages and disadvantages of each traffic type. Let’s start with unicast:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/video-streaming-unicast.png" alt=""><figcaption></figcaption></figure>

Above we have a small network with a video server that is streaming a movie and four hosts who want to watch the movie. Two hosts are on the same LAN, the other two hosts are on another site that is connected through a 30 Mbit WAN link.

A single HD video stream requires 6 Mbps of bandwidth. When we are using unicast, the video server will send the packets to each individual host. With four hosts, it means the video server will be streaming 4x 6Mbps = 24Mbps of traffic.

Each additional host that wants to receive this video stream will put more burden on the video server and requires more bandwidth from the WAN link. Right now we require 2x 6Mbps of bandwidth for H3 and H4. When four more hosts would join on the right side, our WAN link would be completely saturated.

What about the LAN on the left side? If these are Gigabit links then a couple of hosts watching a movie will be no problem. What if there’s 150 users that want to watch the movie? That’s when we start running into issues.

The main problem with unicast traffic is that it **is not scalable**. Are there any advantages? It’s simple since unicast works “out of the box”. You will see that multicast requires some additional protocols to make it work. Also, multicast **only supports UDP traffic** so we can’t use the advantages of TCP like windowing and acknowledgments.

What about broadcast traffic?

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/video-streaming-broadcast.png" alt=""><figcaption></figcaption></figure>

If our video server would broadcast its traffic then the load on the video server will be reduced, it’s only sending the packets once. The problem however is that everyone in the broadcast domain will receive it…whether they like it or not. Another issue with broadcast traffic is that routers do not forward broadcast traffic, it will be dropped.

What about multicast traffic?

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/video-streaming-multicast.png" alt=""><figcaption></figcaption></figure>

Multicast traffic is very efficient. This time we only have two hosts that are interested in receiving the video stream. The video server will only send the packets once, the switches and routers will only forward traffic to the hosts that want to receive it. This reduces the load of the video server and network traffic in general.

When using unicast, each additional host will increase the load and traffic rate. With multicast it will remain the same:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/unicast-vs-multicast-graph.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
What about the Internet? Since multicast is so much more efficient than unicast, large companies like Netflix and Youtube must be using this to stream videos right? Unfortunately multicast on the Internet has never really been implemented. These large video companies use LOTS of unicast traffic to deliver videos to their customers. The only place where you might see multicast on the Internet is your local ISP. They typically use multicast for IPTV to deliver video to their own customers.
{% endhint %}

## Multicast Components

Multicast is efficient but it doesn’t work “out of the box”. There are a number of components that we require:

First of all we use a designated range of IP address that is exclusively used for multicast traffic. We use the class D range for this: 224.0.0.0 to 239.255.255.255. These addresses are only used as **destination addresses**, not as source addresses. The source IP address will be the device that is sending the multicast traffic, for example the video server.

We also require applications that support multicast. A simple example is the [VLC mediaplayer](http://www.videolan.org/vlc/index.html), it can be used to stream and receive a video on the network.

When a router receives multicast traffic, somehow it has to know if anyone is interested in receiving the multicast traffic. Take a look at the picture below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/host-wants-multicast-traffic.png" alt=""><figcaption></figcaption></figure>

Above you can see the router is receiving the multicast traffic from the video server. It doesn’t know _where_ and _if_ it should forward this multicast traffic. We need some mechanism on our hosts that **tell the router when they want to receive multicast traffic**. We use the [**IGMP (Internet Group Management Protocol)**](https://networklessons.com/cisco/ccnp-encor-350-401/igmp-version-1) for this. Hosts that want to receive multicast traffic will use the IGMP protocol to tell the router which multicast traffic they want to receive.

IGMP helps the router to figure out on what interfaces it should forward multicast traffic but what about switches? Take a look at the following image:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/switch-wants-to-forward-multicast-traffic.png" alt=""><figcaption></figcaption></figure>

Our router knows that it has to forward the multicast traffic since a host used IGMP to tell the router it is interested. Once the multicast traffic arrives at the switch, we have another problem. Switches learn MAC addresses by looking at the source address of an Ethernet frame. Since we use multicast addresses only for the destination, how is the switch supposed to learn where to forward multicast traffic to?

To help the switch figure out where to forward multicast traffic, we can use [IGMP snooping](https://networklessons.com/cisco/ccnp-encor-350-401/igmp-snooping). The switch will “listen” to IGMP messages between the host(s) and router to figure out where it should forward multicast traffic to. There’s also a Cisco proprietary protocol called CGMP (Cisco Group Management Protocol) that can be used between switches and routers. The router will then be able to inform the switch where to forward multicast traffic. Unlike IGMP snooping, CGMP isn’t used much.

Last but not least, we need a multicast routing protocol:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/01/multicast-routing-overview.png" alt=""><figcaption></figcaption></figure>

Above we have our video server that is forwarding multicast traffic to R1. On the bottom there’s H1 who is interested in receiving it.

With unicast routing, each router advertises its directly connected interfaces in a routing protocol. Routers who receive unicast packets only care about the **destination address**. They check their routing tables, find the outgoing interface and forward the packets towards the destination. With multicast routing, things are not that simple…the destination is a multicast group address and the multicast packets have to be forwarded to multiple receivers throughout the network.

To accomplish this, we use a multicast routing protocol:

* DVMRP (Distance Vector Multicast Routing Protocol)
* MOSPF (Multicast Open Shortest Path First)
* PIM (Protocol Independent Multicast)

The most popular multicast routing protocol is PIM which we will cover in different lessons.

## Conclusion

In this lesson you have learned the basics of multicast. You have seen the difference between unicast, broadcast and multicast and how multicast is far more scalable than the other two traffic types. We also discussed the different protocols that are required to make multicast work:

* IGMP so hosts can tell routers they want to receive multicast traffic.
* IGMP snooping so the switch knows where to forward multicast traffic.
* Multicast routing: we need a protocol like PIM that can route multicast traffic.

Multicast has many advantages, the main advantage is the scalability compared to unicast traffic. One of the disadvantages is that we require applications that support multicast and we have to configure the network to support it.

In the next lessons, we’ll take a close look at IGMP, PIM and how to configure multicast. I hope you enjoyed this lesson, please share it with your friends. If you have any questions feel free to leave a comment in our forum.
