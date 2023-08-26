# OSPF LSA and LSDB Flooding

How exactly does OSPF fill the LSDB? Let’s zoom in on the operation of how OSPF keeps its link-state database up-to-date:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-lsa-lsdb-flooding.png" alt=""><figcaption></figcaption></figure>

Each LSA has an aging timer that carries the link-state age field. By default, each OSPF LSA is only valid for 30 minutes. If the LSA expires then the router that created the LSA will resend the LSA and increase the sequence number. &#x20;

Let’s walk through this flowchart together. In this example, a new LSA is arriving at the router, and OSPF has to decide what to do with it:

1. If the LSA isn’t already in the LSDB it will be added, and an LSAck (acknowledgment) will be sent to the OSPF neighbor. The LSA will be flooded to all other OSPF neighbors, and we have to run SPF to update our routing table.
2. If the LSA is already in the LSDB and the sequence number is the same, then we will ignore the LSA.
3. If the LSA is already in the LSDB and the sequence number is different, then we have to take action:
   1. If the sequence number is higher it means this information is newer and we have to add it to our LSDB.
   2. If the sequence number is lower it means our OSPF neighbor has an old LSA and we should help them. We will send a LSU (Link state update) including the newer LSA to our OSPF neighbor. The LSU is an envelope that can carry multiple LSAs in it.

It’s not just the sequence number that OSPF will look at to determine if an LSA is more recent. It will consider the LSA to be more recent if it has the following:

* A higher sequence number.
* A higher checksum number.
* If the link-state age is much younger.

What do the sequence numbers look like for OSPF LSAs?

* There are 4 bytes or 32 bits.
* Begins with 0x80000001 and ends at 0x7FFFFFFF.
* Every 30 minutes, each LSA will age out and will be flooded:
  * The sequence number will increment by one.

With 32 bits, we have a LOT of sequence numbers, and every 30 minutes, it will increase. If we reach the last sequence number 0x7FFFFFFF, it will wrap around and start again at 0x80000001. Every 30 minutes OSPF will flood an LSA to ensure the LSDB stays up to date, and when it does this, the sequence number will increase, and OSPF will reset the max-age when it receives a new LSA update.

You can view the OSPF link-state database on a Cisco router using the following command:

<pre><code><strong>Router#show ip ospf database 
</strong>
            OSPF Router with ID (2.2.2.2) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
1.1.1.1         1.1.1.1         109         0x80000002 0x004ED8 2
2.2.2.2         2.2.2.2         108         0x80000002 0x003EDB 2

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
192.168.12.2    2.2.2.2         108         0x80000001 0x008F1F
</code></pre>

Here is the output of the `show ip ospf database` command, which shows us the LSDB. What interesting things can we see here?

* Each OSPF router has interfaces in an area. For each interface, it will advertise a router LSA. The link ID is the OSPF router ID.
* The first router LSA entry you see is from an OSPF router with router ID 1.1.1.1, and you can see the age, sequence number, and checksum. You can see that the LSA has been updated two times since the sequence number is 0x80000002.

That’s all for now. It’s best to look at the OSPF link-state database yourself as you are doing labs. If you have any questions feel free to ask!
