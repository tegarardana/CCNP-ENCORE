# BGP Messages

BGP uses a variety of messages for establishing the connection, exchanging routing information, checking if the remote BGP neighbor is still there and/or notifying the remote side if any errors occur.

To do all of this, BGP uses 4 messages:

* **Open Message**
* **Update Message**
* **Keepalive Message**
* **Notification Message**

All of these BGP messages use a fixed-size header, it includes a type field that indicates what type of message it is.

To explain these BGP messages I will show you some Wireshark captures. I will use the following topology for this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/bgp-r1-r2-as1-as2-topology.png" alt=""><figcaption></figcaption></figure>

### Open Message

Once two BGP routers have completed a TCP 3-way handshake they will attempt to [establish a BGP session](https://networklessons.com/cisco/ccnp-encor-350-401/bgp-neighbor-adjacency-states), this is done using open messages. In the open message you will find some information about the BGP router, these have to be negotiated and accepted by both routers before we can exchange any routing information. Here are some of the items you will find in the open message:

* **Version**: this includes the BGP version that the router is using. The current version of BGP is version 4 which is described in [RFC 4271](http://tools.ietf.org/html/rfc4271). Two BGP routers will try to negotiate a compatible version, when there is a mismatch then there will be no BGP session.
* **My AS**: this includes the AS number of the BGP router, the routers will have to agree on the AS number(s) and it also defines if they will be running iBGP or eBGP.
* **Hold Time**: if BGP doesn’t receive any keepalive or update messages from the other side for the duration of the hold time then it will declare the other side ‘dead’ and it will tear down the BGP session. By default the hold time is set to 180 seconds on Cisco IOS routers, the keepalive message is sent every 60 seconds. BGP routers will use the lowest configured hold down timer.
* **BGP Identifier**: this is the local BGP router ID which is elected just like OSPF does:
  * Use the router-ID that was configured manually with the **bgp router-id** command.
  * Use the highest IP address on a loopback interface.
  * Use the highest IP address on a physical interface.
* **Optional Parameters:** here you will find some optional capabilities of the BGP router. This field has been added so that new features could be added to BGP without having to create a new version.Things you might find here are:
  * support for [MP-BGP (Multi Protocol BGP).](https://networklessons.com/cisco/ccnp-encor-350-401/multiprotocol-bgp-mp-bgp-configuration)
  * support for [Route Refresh](https://networklessons.com/cisco/ccnp-encor-350-401/bgp-route-refresh-capability).
  * support for [4-octet AS numbers](https://networklessons.com/bgp/bgp-private-and-public-as-range).

Here’s an example of a wireshark capture of an open message between R1 and R2:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/wireshark-capture-bgp-open-message.png" alt=""><figcaption></figcaption></figure>

Above you can see the open message from R1 to R2. You can see the things that we discussed, the BGP version, AS number, hold time, BGP ID and the optional parameters (MP-BGP and route refresh). The marker field on top is used to indicate if we use MD5 authentication or not. When it’s filled with 1’s then we are not using authentication.

## Update Message

Once two routers have become BGP neighbors, they can start exchanging routing information. This is done with the update message. In the update message you will find information about the prefixes that are advertised.In “BGP language” a prefix is referred to as **NLRI (Network Layer Reachability Information)**. Here are some of the things you will find in an update message:

* Withdrawn Route Length: this field shows the length of the Withdrawn Routes field in bytes. When it is set to 0, there are no routes withdrawn and the Withdrawn Routes field will not show up.
* Withdrawn Routes: this field shows all the prefixes that should be removed from the BGP table.
* Total Path Attribute Length: here you will find the total length of the Path Attributes field.
* Path Attributes: the BGP attributes for the prefix are stored here, for example: origin, as\_path, next\_hop, med, local preference, etc. These path attributes are stored in TLV-format (Type, Length, Value).

Each of the BGP attributes also has an **attribute flag** that tells the BGP router how to treat the attribute. Here are the different bit flags:

* Optional: when the attribute is well-known this bit is set to 0, when its optional it is set to 1.
* Transitive: when an optional attribute is non-transitive this bit is set to 0, when it is transitive it is set to 1.
* Partial: when an optional attribute is complete this bit is set to 0, when it’s partial it is set to 1.
* Extended Length: when the attribute length is 1 octet it is set to 0, for 2 octets it is set to 1. This extended length flag may only be used if the length of the attribute value is greater than 255 octets.

Let’s take a look at an update message from R1:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 1.1.1.1 mask 255.255.255.255
</strong></code></pre>

Here’s the capture:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/wireshark-capture-bgp-update-route-message.png" alt=""><figcaption></figcaption></figure>

Above you can see a update message from R1. No routes are withdrawn and there are a couple of BGP attributes. You can see the ORIGIN, AS\_PATH and MULTI\_EXIT\_DISC (MED). I also highlighted some of the flags. The AS\_PATH attribute is transitive while MULTI\_EXIT\_DISC is optional. At the bottom you can find the NLRI information with our prefix.

Let’s remove the network command for the loopback interface on R1 so that we can see a withdrawn in the update message:

<pre><code><strong>R1(config)#interface loopback 0
</strong><strong>R1(config-if)#shutdown
</strong></code></pre>

Here’s the capture:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/wireshark-capture-bgp-update-withdrawn-message.png" alt=""><figcaption></figcaption></figure>

Here you can see the withdrawn routes length which is 5 bytes. In the Withdrawn Routes field we see our 1.1.1.1 /32 prefix that should be removed.

## Keepalive Message

When there are no routes to be advertised or withdrawn, there’s not much our BGP neighbors have to share with each other. To make sure the other side is “still there” we use these periodic keepalive messages. By default, BGP sends 19 byte long keepalive messages every 60 seconds. When a remote BGP neighbor misses three keepalives (3 x 60 = 180 seconds, the value of the hold time) it will flush the routes from the BGP neighbor.

Here’s a capture of a keepalive message:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/wireshark-capture-bgp-keepalive-message.png" alt=""><figcaption></figcaption></figure>

The keepalive message is really simple, it’s just a basic header with the length (19 bytes) and the type.

## Notification Message

The notification message is used when an error occurs which will result in termination of the BGP neighbor adjacency. When something goes wrong, the notification message will be sent and the session will be terminated.

The TCP session will be cleared, all entries from this BGP neighbor will be removed from the BGP table and update messages with route withdrawals will be sent to other BGP neighbors.

There is a list with BGP error codes and each error code has a sub-type. Here are some examples:

* Message header error
* Open message error
* Update message error

For each of those there is a subtype that explains the exact error. For example for the open message here are some of the subtypes:

* Unsupported version number
* Bad peer AS
* Bad BGP identifier
* Unsupported optional parameter
* Unacceptable hold time

The list with all error codes and their subtypes is quite large. If you want to see all of them, [take a look at this list from IANA](http://www.iana.org/assignments/bgp-parameters/bgp-parameters.xhtml#bgp-parameters-3).

Let me show you an example of a notification message, we’ll do something that BGP doesn’t like:

<pre><code><strong>R2(config)#no router bgp 2
</strong><strong>R2(config)#router bgp 22
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong></code></pre>

By changing the AS number on one of the routers we will have a mismatch. Here’s the wireshark capture:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/wireshark-capture-bgp-notification-message.png" alt=""><figcaption></figcaption></figure>

R1 is sending R2 a notification message with a major error “open message error” and the minor error code (subtype) is bad peer AS.

[Wireshark Capture eBGP Neighbor Adjacency](https://www.cloudshark.org/captures/89f1795591f6)

These are the messages that BGP uses, I hope this lesson has been useful to you…if you have any questions, just leave a comment!
