# EIGRP Neighbor and Topology Table

The EIGRP topology table is something most networking students find difficult to read. In this lesson, I’ll give you an example of what it looks like and the information you can find in it. We’ll do this by looking at some real routers. Here’s the topology I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-neighbor-adjacency-lab.png" alt=""><figcaption></figcaption></figure>

I’m using two routers with a loopback interface each, and EIGRP has been configured. The routers have become EIGRP neighbors, as we can see here:

<pre><code><strong>R1#show ip eigrp neighbors 
</strong>IP-EIGRP neighbors for process 1
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
0   192.168.12.2            Fa0/0             10 00:06:06   19   200  0  27
</code></pre>

In the output above, I’m looking at the EIGRP neighbor table of R1. As you can see, we have one neighbor (192.168.12.2), which happens to be R2 on interface FastEthernet 0/0. What else do we find here?

* H (Handle): Here, you will find the order when the neighbor adjacency was established. Your first neighbor will have a value of 0, the second neighbor a value of 1, and so on.
* Hold: (sec): this is the hold-down timer per EIGRP neighbor. Once this timer expires, we will drop the neighbor adjacency. The default hold-down timer is 15 seconds. On older IOS versions, only a hello packet would reset the hold-down timer, but on newer IOS versions, any EIGRP packet after the first hello will reset the hold-down timer.
* Uptime: How long the neighbor has been up.
* SRTT (Smooth round-trip time): The number of milliseconds it takes to send an EIGRP packet to your neighbor and receive an acknowledgment packet back.
* RTO (Retransmission timeout): The amount of time in milliseconds that EIGRP will wait before retransmitting a packet from the retransmission queue to this neighbor.
* Q Cnt (Q count): The number of EIGRP packets (Update, Query, or Reply) in the queue that are awaiting transmission. Ideally, you want this number to be 0. Otherwise, it might be an indication of congestion on the network.
* Seq Num (Sequence number): This will show you the sequence number of the last update, query, or reply packet that you received from your EIGRP neighbor.

Excellent, so that’s how EIGRP stores neighbor information! Our next stop is, of course, to take a look at the **EIGRP Topology table**:

<pre><code><strong>R1#show ip eigrp topology 
</strong>IP-EIGRP Topology Table for AS(1)/ID(1.1.1.1)

Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 1.1.1.0/24, 1 successors, FD is 128256
        via Connected, Loopback0
P 2.2.2.0/24, 1 successors, FD is 156160
        via 192.168.12.2 (156160/128256), FastEthernet0/0
P 192.168.12.0/24, 1 successors, FD is 28160
        via Connected, FastEthernet0/0
</code></pre>

Now that’s a lot of information to look at! Let me break it down for you in chunks:

<pre><code><strong>R1#show ip eigrp topology 
</strong>IP-EIGRP Topology Table for AS(1)/ID(1.1.1.1)

Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 1.1.1.0/24, 1 successors, FD is 128256
        via Connected, Loopback0
P 2.2.2.0/24, 1 successors, FD is 156160
        via 192.168.12.2 (156160/128256), FastEthernet0/0
P 192.168.12.0/24, 1 successors, FD is 28160
        via Connected, FastEthernet0/0
</code></pre>

If you look at the red fonts, you can see that we are looking at the EIGRP topology table for AS (Autonomous System) number 1. Remember that the AS number has to match on EIGRP routers to become neighbors! What are the codes about? Let’s see:

<pre><code><strong>R1#show ip eigrp topology 
</strong>IP-EIGRP Topology Table for AS(1)/ID(1.1.1.1)

Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 1.1.1.0/24, 1 successors, FD is 128256
        via Connected, Loopback0
P 2.2.2.0/24, 1 successors, FD is 156160
        via 192.168.12.2 (156160/128256), FastEthernet0/0
P 192.168.12.0/24, 1 successors, FD is 28160
        via Connected, FastEthernet0/0
</code></pre>

Look at those codes…Update, Query, and Reply should ring a bell since I discussed them a few lessons ago. Let’s focus on those codes that I didn’t explain before…

* Passive: Passive is good…we like routing information to be passive, which means that we have learned information about a network and there are no changes in the topology table.\

* Active: Active is not good since it means we have lost information about a certain network, and EIGRP doesn’t know another way of reaching this network. It will go into active mode and send query packets to ALL its neighbors, asking them if they know how to reach this network.
* Reply Status: EIGRP will track all the query packets it has sent to neighbors since you need a reply in return. By setting the reply status flag, it will do this.
* Sia Status (Stuck in Active): This is a bad one…it means that EIGRP has not received a reply to a query packet from one of the neighbors within the allowed time (about 3 minutes). When this happens, EIGRP will drop the neighbor adjacency, and it will be stuck in active. More on this later!

Let’s take a closer look at one entry in the topology table:

<pre><code><strong>R1#show ip eigrp topology 
</strong>IP-EIGRP Topology Table for AS(1)/ID(1.1.1.1)

Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 1.1.1.0/24, 1 successors, FD is 128256
        via Connected, Loopback0
P 2.2.2.0/24, 1 successors, FD is 156160
        via 192.168.12.2 (156160/128256), FastEthernet0/0
P 192.168.12.0/24, 1 successors, FD is 28160
        via Connected, FastEthernet0/0
</code></pre>

Let’s look at an entry of a prefix…in this case, 2.2.2.0/24 and break it down into pieces:

* P 2.2.2.0/24: The P stands for passive, and that’s how we like it! As you can see, EIGRP stores the network and the subnet mask.
* 1 successor: The best path to get to a certain network is called the successor. It’s possible to have backup paths, which EIGRP calls the feasible successor. In this case, there is only one way to get to the destination.
* FD is 156160: FD stands for feasible distance, and in plain English, we would call this the “total distance” to get to the destination.
* Via 192.168.12.2: That’s the IP address of the neighbor to which we have to send packets to reach the 2.2.2.0/24 network.
* (156160/128256): The first value is the feasible distance. The second value is the advertised distance. Your EIGRP neighbor will report to you how far it is for him to reach the 2.2.2.0/24 network, which is saved as the advertised distance.
* FastEthernet0/0: This one is easy; it’s just the interface we are using to send our packets to to reach this network.

Does this make sense to you? Understanding the EIGRP topology table is very important when you are troubleshooting missing routes, and you’ll have to look at it when you are trying to configure EIGRP unequal cost load balancing. If you have any questions feel free to leave a comment!
