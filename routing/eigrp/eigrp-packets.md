# EIGRP Packets

In this lesson I want to explain the different EIGRP packets to you.

Hello packets are sent between EIGRP neighbors for neighbor discovery and recovery. If you send hello packets and receive them then EIGRP will form a neighbor relationship with the other router. As long as you receive hello packets from the other side, EIGRP will believe that the other router is still there. As soon as you don’t receive them anymore you will drop the neighbor relationship called adjacency, and EIGRP might have to look for another path for certain destinations.

EIGRP uses RTP (Reliable Transport Protocol), and its function is to deliver EIGRP packets between neighbors in a reliable and ordered way. It can use multicast or unicast and to keep things efficient, not all packets are sent reliably. Reliable means that when we send a packet we want to get an acknowledgment from the other side to make sure that they received it. So when does EIGRP use unicast or multicast? Let’s take a look at an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-hello-packet-overhead.png" alt=""><figcaption></figcaption></figure>

In this example, we have four routers all running EIGRP. Hello packets are sent between routers to form adjacencies. As you can see, R1 is sending three hello packets meant for R2, R3, and R4. There are two questions that we can ask ourselves here:

* Is it really useful to send 3 different hello packets on a single link?
* Is it necessary that a hello packet gets an acknowledgment in return?

Sending 3 packets on the same link is not very useful, so instead of doing this, EIGRP will send hello packets by using multicast on a multi-access network like Ethernet.

Hello packets don’t have to be acknowledged since EIGRP uses a hold-down time. If a router doesn’t receive hello packets in an X amount of time, it will drop the neighbor adjacency.

So which packets should be acknowledged? Think about routing information, if there’s a change in the network, you want to make sure all routers receive this routing update.

Let me show you all the different EIGRP packets:

* Hello
* Update
* Query
* Reply
* ACK (Acknowledgement)

Hello packets are used for neighbor discovery. As soon as you send hello packets and receive them, your EIGRP routers will try to form the neighbor adjacency.

Update packets have routing information and are sent reliably to whatever router that requires this information. Update packets can be sent to a single neighbor using unicast or to a group of neighbors using multicast.

Query packets are used when your EIGRP router has lost information about a certain network and doesn’t have any backup paths. What happens is that your router will send query packets to its neighbors, asking them if they have information about this particular network.

Reply packets are used in response to the query packets and are reliable.

ACK packets are used to acknowledge the receipt of update, query, and reply packets. ACK packets are sent by using unicast.

That’s all for now. I hope you now have a better understanding of the different packets that EIGRP uses. If you have any more questions, leave a comment!
