# OSPF Packets and Neighbor Discovery

In this lesson, I’m going to show you the different packets OSPF uses and how neighbor discovery works.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-header.png" alt=""><figcaption></figcaption></figure>

OSPF uses its own protocol, like EIGRP, and doesn’t use a transport protocol like TCP or UDP. If you would look at the IP packet in Wireshark, you can see that OSPF has protocol ID 89 for all its packets.

<pre><code><strong>R2#debug ip ospf packet 
</strong>OSPF packet debugging is on
OSPF: rcv. v:2 t:1 l:48 rid:1.1.1.1
      aid:0.0.0.0 chk:4D40 aut:0 auk: from FastEthernet0/0
</code></pre>

If we use `debug ip ospf packet` we can look at the OSPF packet on our router. Let’s look at the different fields we have:

* V:2 stands for OSPF version 2. If you are running IPv6, you’ll version 3.
* T:1 stands for OSPF packet number 1, which is a hello packet. I’m going to show you the different packets in a bit.
* L:48 is the packet length in bytes. This hello packet seems to be 48 bytes.
* RID 1.1.1.1 is the Router ID.
* AID is the area ID in dotted decimal. You can write the area in decimal (area 0) or dotted decimal (area 0.0.0.0).
* CHK 4D40 is the checksum of this OSPF packet, so we can check if the packet is corrupt or not.
* AUT:0 is the authentication type. You have three options:
  * 0 = no authentication
  * 1 = clear text
  * 2 = MD5
  * AUK: If you enable authentication, you’ll see some information here.

Let’s continue by looking at the different OSPF packet types:

<div align="left">

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-packets.png" alt=""><figcaption></figcaption></figure>

</div>

I’m throwing them right at you; here are all our OSPF packet types. What is the role of each OSPF packet?

* Hello: neighbor discovery, build neighbor adjacencies and maintain them.
* DBD: This packet is used to check if the LSDB between the two routers is the same. The DBD is a summary of the LSDB.
* LSR: Requests specific link-state records from an OSPF neighbor.
* LSU: Sends specific link-state records that were requested. This packet is like an envelope with multiple LSAs in it.
* LSAck: OSPF is a reliable protocol, so we have a packet to acknowledge the others.

OSPF has to get through 7 states to become neighbors…here they are:

1. Down: no OSPF neighbors have been detected at this moment.
2. Init: Hello packet received.
3. Two-way: own router ID found in received hello packet.
4. Exstart: master and slave roles determined.
5. Exchange: database description packets (DBD) are sent.
6. Loading: exchange of LSRs (Link state request) and LSUs (Link state update) packets.
7. Full: OSPF routers now have an adjacency.

Let’s have a detailed look at this process! You can also see it in this packet capture:

[OSPF Neighbor Adjacency – Network Type Broadcast](https://www.cloudshark.org/captures/111cb2076caa)

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/router-r1-r2.png" alt=""><figcaption></figcaption></figure>

This is the topology I’m using. R1 and R2 are connected using a single link, and we will see how R1 learns about the 2.2.2.0 /24 network.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-down-state.png" alt=""><figcaption></figcaption></figure>

As soon as I configure OSPF on R1, it will send hello packets. R1 has no clue about other OSPF routers at this moment, so it’s in the down state. The hello packet will be sent to the multicast address 224.0.0.5.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-init-state.png" alt=""><figcaption></figcaption></figure>

R2 receives the hello packet and will put an entry for R1 in the OSPF neighbor table. We are now in the init state.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-two-way-state.png" alt=""><figcaption></figcaption></figure>

R2 has to respond to R1 with a hello packet. This packet is not sent using multicast but with unicast, and in the neighbor field, it will include all OSPF neighbors that R2 has. R1 will see **its** own name in the neighbor field in this hello packet.

R1 will receive this hello packet and sees its own router ID. We are now in the two-way state.

I have to take a pause here. If the link we use is a multi-access network, OSPF has to elect a DR (Designated Router) and BDR (Backup Designated Router). This has to happen before we can continue with the rest of the process. In another lesson, I’m going to teach you DR/BDR…for now, just hold the thought that the DR/BDR election happens right after the two-way state, ok?

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-exstart-state-1.png" alt=""><figcaption></figcaption></figure>

Our next stop is the exstart state. Our routers are ready to sync their LSDB. At this step, we have to select a master and slave role. The router with the highest router ID will become the master. R2 has the highest router ID and will become the master.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-exchange-state.png" alt=""><figcaption></figcaption></figure>

In the exchange state, our routers are sending a DBD with a summary of the LSDB. This way, the routers can find out what networks they don’t know about.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-loading-state.png" alt=""><figcaption></figcaption></figure>

When our routers receive the DBD from the other side, they will do a couple of things:

* Send an acknowledgment using the LSAck packet.
* Compare the information in the DBD with the information it already has:
  * If the neighbor has new or newer information, it will send an LSR (Link State Request) packet to request this information
  * When the routers start sending an LSR (Link State Request) we are in the loading state.
  * The other router will respond with an LSU (Link State Update) with the requested information.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-full-state.png" alt=""><figcaption></figcaption></figure>

When R1 requested information about 2.2.2.0 /24, it used an LSR. R2 will send the LSU with the information. R1 will send an acknowledgment using an LSAck packet to finish. We are now in the full state. Both routers have a synchronized LSDB, and we are ready to route!

Wonder what this looks like on a real router? Doing a `debug ip ospf adj` is faster than drawing all those pictures in Microsoft Visio:

<pre><code><strong>R2#debug ip ospf adj 
</strong>OSPF adjacency events debugging is on
</code></pre>

<pre><code><strong>R2#clear ip ospf process 
</strong>Reset ALL OSPF processes? [no]: yes
</code></pre>

Let me show you the debug:

```
R2#
OSPF: Interface Loopback0 going Down
OSPF: 2.2.2.2 address 2.2.2.2 on Loopback0 is dead, state DOWN
OSPF: Interface FastEthernet0/0 going Down
OSPF: 2.2.2.2 address 192.168.12.2 on FastEthernet0/0 is dead, state DOWN
OSPF: Neighbor change Event on interface FastEthernet0/0
OSPF: DR/BDR election on FastEthernet0/0 
OSPF: Elect BDR 0.0.0.0
OSPF: Elect DR 1.1.1.1
OSPF: Elect BDR 0.0.0.0
OSPF: Elect DR 1.1.1.1
       DR: 1.1.1.1 (Id)   BDR: none 
OSPF: 1.1.1.1 address 192.168.12.1 on FastEthernet0/0 is dead, state DOWN
%OSPF-5-ADJCHG: Process 1, Nbr 1.1.1.1 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: Interface down or detached
OSPF: Neighbor change Event on interface FastEthernet0/0
OSPF: DR/BDR election on FastEthernet0/0 
OSPF: Elect BDR 0.0.0.0
OSPF: Elect DR 0.0.0.0
       DR: none    BDR: none 
OSPF: Remember old DR 1.1.1.1 (id)
OSPF: Interface Loopback0 going Up
OSPF: Interface FastEthernet0/0 going Up
OSPF: 2 Way Communication to 1.1.1.1 on FastEthernet0/0, state 2WAY
OSPF: Backup seen Event before WAIT timer on FastEthernet0/0
OSPF: DR/BDR election on FastEthernet0/0 
OSPF: Elect BDR 2.2.2.2
OSPF: Elect DR 1.1.1.1
OSPF: Elect BDR 2.2.2.2
OSPF: Elect DR 1.1.1.1
       DR: 1.1.1.1 (Id)   BDR: 2.2.2.2 (Id)
OSPF: Send DBD to 1.1.1.1 on FastEthernet0/0 seq 0x1E09 opt 0x52 flag 0x7 len 32
OSPF: Rcv DBD from 1.1.1.1 on FastEthernet0/0 seq 0x886 opt 0x52 flag 0x7 len 32  mtu 1500 state EXSTART
OSPF: First DBD and we are not SLAVE
OSPF: Rcv DBD from 1.1.1.1 on FastEthernet0/0 seq 0x1E09 opt 0x52 flag 0x2 len 72  mtu 1500 state EXSTART
OSPF: NBR Negotiation Done. We are the MASTER
OSPF: Send DBD to 1.1.1.1 on FastEthernet0/0 seq 0x1E0A opt 0x52 flag 0x1 len 32
OSPF: Rcv DBD from 1.1.1.1 on FastEthernet0/0 seq 0x1E0A opt 0x52 flag 0x0 len 32  mtu 1500 state EXCHANGE
OSPF: Exchange Done with 1.1.1.1 on FastEthernet0/0
OSPF: Send LS REQ to 1.1.1.1 length 24 LSA count 2
OSPF: Rcv LS UPD from 1.1.1.1 on FastEthernet0/0 length 108 LSA count 2
OSPF: Synchronized with 1.1.1.1 on FastEthernet0/0, state FULL
%OSPF-5-ADJCHG: Process 1, Nbr 1.1.1.1 on FastEthernet0/0 from LOADING to FULL, Loading Done
```

I highlighted some of the fields: you can see the 2-way communication, the struggle for power to determine who will be master or slave, the exchange of the LSDB summary using another DBD packet, and finally, an LSQ and LSU.

And that’s the end of this lesson. You have now learned about the different OSPF packets and how it forms neighbor adjacencies.

\
