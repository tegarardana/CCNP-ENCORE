# BGP Neighbor Adjacency States

Just like OSPF or EIGRP, BGP establishes a neighbor adjacency with other BGP routers before they exchange any routing information. Unlike other routing protocols however, BGP does not use broadcast or multicast to “discover” other BGP neighbors.

Neighbors have to be configured **manually** and BGP uses **TCP port 179** for the connection.

In this lesson we’ll take a close look at the different “states” when two BGP routers try to become neighbors. Here they are:

1. **Idle**:This is the first state where BGP waits for a “start event”. The start event occurs when someone configures a new BGP neighbor or when we reset an established BGP peering. After the start event, BGP will initialize some resources, resets a ConnectRetry timer and initiates a TCP connection to the remote BGP neighbor. It will also start listening for a connection in case the remote BGP neighbor tries to establish a connection. When successful, BGP moves to the Connect state. When it fails, it will remain in the Idle state.
2. **Connect**: BGP is waiting for the TCP three-way handshake to complete. When it is successful, it will continue to the OpenSent state. In case it fails, we continue to the Active state. If the ConnectRetry timer expires then we will remain in this state. The ConnectRetry timer will be reset and BGP will try a new TCP three-way handshake. If anything else happens (for example resetting BGP) then we move back to the Idle state.
3. **Active**: BGP will try another TCP three-way handshake to establish a connection with the remote BGP neighbor. If it is successful, it will move to the OpenSent state. If the ConnectRetry timer expires then we move back to the Connect state. BGP will also keep listening for incoming connections in case the remote BGP neighbor tries to establish a connection. Other events can cause the router to go back to the Idle state (resetting BGP for example).
4. **OpenSent**: In this state BGP will be waiting for an Open message from the remote BGP neighbor. The Open message will be checked for errors, if something is wrong (incorrect version numbers, wrong AS number, etc.) then BGP will respond with a Notification message and jumps back to the Idle state. This is also the moment where BGP decides whether we use EBGP or IBGP (since we check the AS number). If everything is OK then BGP starts sending keepalive messages and resets its keepalive timer. At this moment, the hold time is negotiated (lowest value is picked) between the two BGP routers. In case the TCP session fails, BGP will jump back to the Active state. When any other errors occur (expiration of hold timer), BGP will send a notification message with the error code and jumps back to the Idle state. In case someone resets the BGP process, we also jump back to the Idle state.
5. **OpenConfirm**: BGP waits for a keepalive message from the remote BGP neighbor. When we receive the keepalive, we can move to the established state and the neighbor adjacency will be completed. When this occurs, it will reset the hold timer. If we receive a notification message from the remote BGP neighbor then we fall back to the Idle state. BGP will keep sending keepalive messages.
6. **Established**: The BGP neighbor adjacency is complete and the BGP routers will send update packets to exchange routing information. Every time we receive a keepalive or update message, the hold timer will be resetted. In case we receive a notification message we will jump back to the Idle state.

This whole process of becoming BGP neighbors can be visualized, this might be a bit easier then just reading about it. The official name of a “diagram” that shows the different states and we can move from one state to another is called a FSM (Finite State Machine). For BGP, it looks like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/bgp-states-neighbor-adjacency.png" alt=""><figcaption></figcaption></figure>

Now you know about the different states, let’s take a look at some Cisco BGP routers to see what it actually looks like on two routers. I’ll use the following topology for this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/bgp-r1-r2-as1-as2.png" alt=""><figcaption></figcaption></figure>

Just two routers in two different autonomous systems. Before I configure BGP, let’s enable a debug:

<pre><code><strong>R1, R2 #debug ip bgp
</strong>BGP debugging is on for address family: IPv4 Unicast
</code></pre>

This will give us everything…

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong></code></pre>

As soon as I do this you will see some debugging;

```
R1#
BGP: 192.168.12.2 active went from Idle to Active
BGP: 192.168.12.2 open active, local address 192.168.12.1
BGP: 192.168.12.2 open failed: Connection refused by remote host
BGP: 192.168.12.2 Active open failed - tcb is not available, open active delayed 9216ms (35000ms max, 60% jitter)
BGP: ses global 192.168.12.2 (0x4B43F3FC:0) act Reset (Active open failed).
BGP: 192.168.12.2 active went from Active to Idle
BGP: nbr global 192.168.12.2 Active open failed - open timer running
```

As soon as I configure BGP on R1 it will try to connect to R2. You can see the debug says that the state moves from Idle to Active (it doesn’t show the Connect state in the debug). When it fails, it falls back to the Idle state. Now let’s configure BGP on R2 as well so we can see a successful progress through the states:

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong></code></pre>

Now look at the debug on R1:

```
R1#
BGP: 192.168.12.2 active went from Idle to Active
BGP: 192.168.12.2 open active, local address 192.168.12.1
BGP: ses global 192.168.12.2 (0x4B43F3FC:0) act Adding topology IPv4 Unicast:base
BGP: ses global 192.168.12.2 (0x4B43F3FC:0) act Send OPEN
BGP: 192.168.12.2 active went from Active to OpenSent
BGP: 192.168.12.2 active sending OPEN, version 4, my as: 1, holdtime 180 seconds, ID C0A80C01
BGP: 192.168.12.2 active rcv message type 1, length (excl. header) 34
BGP: ses global 192.168.12.2 (0x4B43F3FC:0) act Receive OPEN
BGP: 192.168.12.2 active rcv OPEN, version 4, holdtime 180 seconds
BGP: 192.168.12.2 active rcv OPEN w/ OPTION parameter len: 24
BGP: 192.168.12.2 active rcvd OPEN w/ optional parameter type 2 (Capability) len 6
BGP: 192.168.12.2 active OPEN has CAPABILITY code: 1, length 4
BGP: 192.168.12.2 active OPEN has MP_EXT CAP for afi/safi: 1/1
BGP: 192.168.12.2 active rcvd OPEN w/ optional parameter type 2 (Capability) len 2
BGP: 192.168.12.2 active OPEN has CAPABILITY code: 128, length 0
BGP: 192.168.12.2 active OPEN has ROUTE-REFRESH capability(old) for all address-families
BGP: 192.168.12.2 active rcvd OPEN w/ optional parameter type 2 (Capability) len 2
BGP: 192.168.12.2 active OPEN has CAPABILITY code: 2, length 0
BGP: 192.168.12.2 active OPEN has ROUTE-REFRESH capability(new) for all address-families
BGP: 192.168.12.2 active rcvd OPEN w/ optional parameter type 2 (Capability) len 6
BGP: 192.168.12.2 active OPEN has CAPABILITY code: 65, length 4
BGP: 192.168.12.2 active OPEN has 4-byte ASN CAP for: 2
BGP: nbr global 192.168.12.2 neighbor does not have IPv4 MDT topology activated
BGP: 192.168.12.2 active rcvd OPEN w/ remote AS 2, 4-byte remote AS 2
BGP: 192.168.12.2 active went from OpenSent to OpenConfirm
BGP: 192.168.12.2 active went from OpenConfirm to Established
BGP: ses global 192.168.12.2 (0x4B43F3FC:1) act Assigned ID
BGP: ses global 192.168.12.2 (0x4B43F3FC:1) Up
%BGP-5-ADJCHANGE: neighbor 192.168.12.2 Up
```

Above you can see that the BGP state moves from Idle to Active and then to OpenSent. Some Open messages are sent and received, the BGP routers are exchanging some of their capabilities. From there we move to the OpenConfirm and Established state. Finally you see the BGP neighbor as up. On R2 we see something similar:

```
R2#
BGP: 192.168.12.1 passive open to 192.168.12.2
BGP: 192.168.12.1 passive went from Idle to Connect
BGP: ses global 192.168.12.1 (0x4B269374:0) pas Setting open delay timer to 60 seconds.
BGP: ses global 192.168.12.1 (0x4B269374:0) pas read request no-op
BGP: 192.168.12.1 passive rcv message type 1, length (excl. header) 34
BGP: ses global 192.168.12.1 (0x4B269374:0) pas Receive OPEN
BGP: 192.168.12.1 passive rcv OPEN, version 4, holdtime 180 seconds
BGP: 192.168.12.1 passive rcv OPEN w/ OPTION parameter len: 24
BGP: 192.168.12.1 passive rcvd OPEN w/ optional parameter type 2 (Capability) len 6
BGP: 192.168.12.1 passive OPEN has CAPABILITY code: 1, length 4
BGP: 192.168.12.1 passive OPEN has MP_EXT CAP for afi/safi: 1/1
BGP: 192.168.12.1 passive rcvd OPEN w/ optional parameter type 2 (Capability) len 2
BGP: 192.168.12.1 passive OPEN has CAPABILITY code: 128, length 0
BGP: 192.168.12.1 passive OPEN has ROUTE-REFRESH capability(old) for all address-families
BGP: 192.168.12.1 passive rcvd OPEN w/ optional parameter type 2 (Capability) len 2
BGP: 192.168.12.1 passive OPEN has CAPABILITY code: 2, length 0
BGP: 192.168.12.1 passive OPEN has ROUTE-REFRESH capability(new) for all address-families
BGP: 192.168.12.1 passive rcvd OPEN w/ optional parameter type 2 (Capability) len 6
BGP: 192.168.12.1 passive OPEN has CAPABILITY code: 65, length 4
BGP: 192.168.12.1 passive OPEN has 4-byte ASN CAP for: 1
BGP: nbr global 192.168.12.1 neighbor does not have IPv4 MDT topology activated
BGP: 192.168.12.1 passive rcvd OPEN w/ remote AS 1, 4-byte remote AS 1
BGP: ses global 192.168.12.1 (0x4B269374:0) pas Adding topology IPv4 Unicast:base
BGP: ses global 192.168.12.1 (0x4B269374:0) pas Send OPEN
BGP: 192.168.12.1 passive went from Connect to OpenSent
BGP: 192.168.12.1 passive sending OPEN, version 4, my as: 2, holdtime 180 seconds, ID C0A80C02
BGP: 192.168.12.1 passive went from OpenSent to OpenConfirm
BGP: 192.168.12.1 passive went from OpenConfirm to Established
BGP: ses global 192.168.12.1 (0x4B269374:1) pas Assigned ID
BGP: nbr global 192.168.12.1 Stop Active Open timer as all topologies are allocated
BGP: ses global 192.168.12.1 (0x4B269374:1) Up
%BGP-5-ADJCHANGE: neighbor 192.168.12.1 Up
```

The output of these debug messages are nice and easy to read. If for some reason your neighbor adjacency doesn’t appear, these debugs can be helpful to solve the problem.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface fastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
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
!
end
```
{% endtab %}
{% endtabs %}

I hope this has been useful to you to understand the different BGP states. If you have any questions, feel free to leave a comment!
