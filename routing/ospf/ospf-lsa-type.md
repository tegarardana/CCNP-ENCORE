# OSPF LSA Type

OSPF uses a LSDB (link state database) and fills this with LSAs (link state advertisement). Instead of using 1 LSA packet OSPF has many different types of LSAs and in this lesson I’m going to show all of them to you. Let’s start with an overview:

* LSA Type 1:            Router LSA
* LSA Type 2:            Network LSA
* LSA Type 3:            Summary LSA
* LSA Type 4:            Summary ASBR LSA
* LSA Type 5:            Autonomous system external LSA
* LSA Type 6:            Multicast OSPF LSA
* LSA Type 7:            Not-so-stubby area LSA
* LSA Type 8:            External attribute LSA for BGP

<div align="left">

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/jigsaw-piece-150x150.png" alt=""><figcaption></figcaption></figure>

</div>

For many people it helps to visualize things in order to understand and remember. I like to visualize OSPF LSAs as jigsaw puzzle pieces. One jigsaw is nothing but all of them together give us the total picture…for OSPF this is the LSDB.

&#x20;

Here’s the first LSA Type:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/ospf-lsa-type-1.png" alt=""><figcaption></figcaption></figure>

Each router within the area will flood a **type 1 router LSA** within the area. In this LSA you will find a list with all the directly connected links of this router. How do we identify a link?

* The IP prefix on an interface.
* The link type. There are 4 different link types:

<table><thead><tr><th width="121.33333333333331">Link Type</th><th>Description</th><th>Link ID</th></tr></thead><tbody><tr><td>1</td><td>Point-to-point connection to another router.</td><td>Neighbor router ID</td></tr><tr><td>2</td><td>Connection to transit network.</td><td>IP address of DR</td></tr><tr><td>3</td><td>Connection to stub network.</td><td>IP Network</td></tr><tr><td>4</td><td>Virtual Link</td><td>Neighbor router ID</td></tr></tbody></table>

Don’t worry too much about the link types for now, we will see them later. Keep in mind that the router LSA **always stays within the area**.

The second LSA type (network LSA) is created for multi-access networks:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/ospf-lsa-type-2.png" alt=""><figcaption></figcaption></figure>

The **network LSA** or **type 2** is created for each multi-access network. Remember the OSPF network types? The [broadcast](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-broadcast-network-type-over-frame-relay) and [non-broadcast](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-point-to-multipoint-non-broadcast-network-type-over-frame-relay) network types require a DR/BDR. If this is the case you will see these network LSAs being generated by the DR. In this LSA we will find all the routers that are connected to the multi-access network, the DR and of course the prefix and subnet mask.

In my example above we will find R1, R2 and the DR in the network LSA. We will also see the prefix 192.168.123.0 /24 in this LSA. Last thing to mention: the network LSA always **stays within the area**.

Let’s look at the third LSA type:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/ospf-lsa-type-3.png" alt=""><figcaption></figcaption></figure>

Type 1 router LSAs **always stay within the area**. OSPF however works with multiple areas and you probably want full connectivity within all of the areas. R1 is flooding a router LSA within the area so R2 will store this in its LSDB. R3 and R4 also need to know about the networks in Area 2.

R2 is going to create a **Type 3 summary LSA** and flood it into area 0. This LSA will flood into all the other areas of our OSPF network. This way all the routers in other areas will know about the prefixes from **other areas.**

The name “summary” LSA is very misleading. By default OSPF is **not going to summarize** anything for you. There is however a command that let you summarize inter-area routes. Take a look at my [OSPF summarization lesson](https://networklessons.com/cisco/ccnp-encor-350-401/how-to-configure-ospf-summarization)if you are interested. If you are looking at the routing table of an OSPF router and see some **O IA** entries you are looking at LSA type 3 summary LSAs. Those are your inter-area prefixes!

Time for the fourth LSA type:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/ospf-lsa-type-4.png" alt=""><figcaption></figcaption></figure>

In this example we have R1 that is redistributing information from the RIP router into OSPF. This makes R1 an **ASBR (Autonomous System Border Router).** What happens is that R1 will flip a bit in the router LSA to identify itself as an ASBR. When R2 who is an ABR receives this router LSA it will create a **type 4 summary ASBR LSA** and flood it into area 0. This LSA will also be flooded in all other areas and is required so all OSPF routers know where to find the ASBR.

What about LSA type 5? Let’s check it out:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/ospf-lsa-type-5.png" alt=""><figcaption></figcaption></figure>

Same topology but I’ve added a prefix (5.5.5.0 /24) at our RIP router. This prefix will be redistributed into OSPF. R1 (our ASBR) will take care of this and create a **type 5 external LSA** for this. Don’t forget we still need type 4 summary ASBR LSA to locate R1. If you ever tried redistribution with OSPF you might have seen **O E1 or E2** entries. Those are the external prefixes and our type 5 LSAs.

What about OSPF LSA type 6? **Type 6 multicast ospf LSA** I can skip because it’s not being used. It’s not even supported by Cisco. We use PIM (Protocol Independent Multicast) for multicast configurations.

If you are studying the LSA types for CCNA R\&S then you don’t have to worry about LSA type 7. These are used for a special area type called NSSA that is only covered in the CCNP ROUTE material.

Let’s look at the last LSA type, number 7:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/ospf-lsa-type-7.png" alt=""><figcaption></figcaption></figure>

Last LSA type…promised! [NSSA areas](https://networklessons.com/ospf/how-to-configure-ospf-nssa-not-so-stubby-area) do not allow type 5 external LSAs. In my picture R1 is still our ASBR redistributing information from RIP into OSPF.

Since type 5 is not allowed we have to think of something else. That’s why we have a **type 7 external LSA** that carries the exact same information but is not blocked within the NSSA area. R2 will translate this type 7 into a type 5 and flood it into the other areas.

Let me summarize the LSA types for you:

* **Type 1 – Router LSA:** The Router LSA is generated by each router for each area it is located. In the link-state ID you will find the originating router’s ID.
* **Type 2 – Network LSA:** Network LSAs are generated by the DR. The link-state ID will be the interface IP address of the DR.
* **Type 3 – Summary LSA:** The summary LSA is created by the ABR and flooded into other areas.
* **Type 4 – Summary ASBR LSA:** Other routers need to know where to find the ASBR. This is why the ABR will generate a summary ASBR LSA which will include the router ID of the ASBR in the link-state ID field.
* **Type 5 – External LSA:** also known as autonomous system external LSA: The external LSAs are generated by the ASBR.
* **Type 6 – Multicast LSA:** Not supported and not used.
* **Type 7 – External LSA:** also known as not-so-stubby-area (NSSA) LSA: As you can see area 2 is a NSSA (not-so-stubby-area) which doesn’t allow external LSAs (type 5). To overcome this issue we are generating type 7 LSAs instead.

The only thing left to do is take a look at these LSAs in action…time to configure some routers!

## Verification

We can see all the OSPF types in the LSDB, to demonstrate this I will use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/ospf-two-areas-3-routers.png" alt=""><figcaption></figcaption></figure>

It’s a simple setup with 3 routers and 2 areas. I’ve added a couple of loopbacks so we have prefixes to look at. Here’s the configuration:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong><strong>R1(config-router)#network 1.1.1.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong><strong>R2(config-router)#network 192.168.23.0 0.0.0.255 area 1
</strong></code></pre>

<pre><code><strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#network 192.168.23.0 0.0.0.255 area 1
</strong><strong>R3(config-router)#network 3.3.3.0 0.0.0.255 area 1
</strong></code></pre>

Let’s start by looking at the LSDB of R1:

<pre><code><strong>R1#show ip ospf database 
</strong>
            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
1.1.1.1         1.1.1.1         30          0x80000003 0x004CD9 2
2.2.2.2         2.2.2.2         31          0x80000002 0x0048E9 1

                Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
192.168.12.2    2.2.2.2         31          0x80000001 0x008F1F

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
3.3.3.3         2.2.2.2         17          0x80000001 0x00D650
192.168.23.0    2.2.2.2         66          0x80000001 0x00A70C
</code></pre>

By using the show ip ospf database we can look at the LSDB and we can see the type 1 router LSAs, type 2 network LSAs and the type 3 summary LSAs here. What else do we find here?

* Link ID: This is what identifies each LSA.
* ADV router: the router that is advertising this LSA.
* Age: The maximum age counter in seconds. The maximum is 3600 seconds or 1 hour.
* Seq#: Here you see the sequence number which starts at 0x80000001 and will increase by 1 for each update.
* Checksum: There is a checksum for each LSA.
* Link count: This will show the total number of directly connected links and is only used for the router LSA.

So that’s LSA type 1,2 and 3. To show you number 4 and 5 I have to make some changes:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/ospf-two-areas-3-routers-abr-asbr.png" alt=""><figcaption></figcaption></figure>

To accomplish this I will redistribute something on R1 into OSPF.

<pre><code><strong>R1(config)#interface loopback 1
</strong><strong>R1(config-if)#ip address 11.11.11.11 255.255.255.0
</strong><strong>R1(config-if)#exit
</strong><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#redistribute connected subnets
</strong></code></pre>

I created an additional loopback interface and configured an IP address. Then I’m telling OSPF to redistribute the directly connected interfaces into OSPF. Let’s look at the LSDB of R2 and R3:

<pre><code><strong>R2#show ip ospf database | begin Type-5
</strong>		Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
11.11.11.0      1.1.1.1     36          0x80000001 0x000F44 0
</code></pre>

Here you can see the type 5 external LSA in the LSDB of R2. Let’s look at R3:

<pre><code><strong>R3#show ip ospf database | begin Summary
</strong>		Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
1.1.1.1         2.2.2.2         149         0x80000001 0x0033FB
192.168.12.0    2.2.2.2         195         0x80000001 0x00219D

		Summary ASB Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
1.1.1.1         2.2.2.2         62          0x80000001 0x004DB9

		Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
11.11.11.0      1.1.1.1         68          0x80000001 0x000F44 0
</code></pre>

R3 is in another area than R1 so it needs to know where to find the ASBR. In the LSDB you can see the type 5 external LSA but also the type 4 summary ASBR LSA which is the address of R1. Because of this LSA, R3 knows how to reach the ASBR. This type 4 LSA is being generated by R2 which is the ABR.

There’s only one more LSA type to show you and that’s number 7. I’ll have to use the NSSA area type for this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/08/ospf-two-areas-nssa-r3-asbr.png" alt=""><figcaption></figcaption></figure>

Area 1 will be an NSSA and I have added an additional loopback on R3. Here’s the configuration:

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#area 1 nssa
</strong></code></pre>

<pre><code><strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#area 1 nssa
</strong></code></pre>

Now I’m going to create an additional loopback interface on R3 and redistribute it into OSPF.

<pre><code><strong>R3(config)#interface loopback 1
</strong><strong>R3(config-if)#ip address 33.33.33.33 255.255.255.0
</strong><strong>R3(config-if)#exit 
</strong><strong>R3(config)#router ospf 1
</strong><strong>R3(config-router)#redistribute connected subnets
</strong></code></pre>

Let’s see what happens to our LSDB:

<pre><code><strong>R3#show ip ospf database | begin Type-7
</strong>		Type-7 AS External Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Tag
33.33.33.0      33.33.33.33     33          0x80000001 0x005F43 0
</code></pre>

You can see R3 has generated a type 7 external LSA for the prefix on my loopback interface.

<pre><code><strong>R2#show ip ospf database | begin Type-7
</strong>		Type-7 AS External Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Tag
33.33.33.0      33.33.33.33     61          0x80000001 0x005F43 0

		Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
11.11.11.0      1.1.1.1         220         0x80000001 0x000F44 0
33.33.33.0      2.2.2.2         54          0x80000001 0x00998F 0
</code></pre>

R2 has the type 7 external LSA in its LSDB since it’s in the same area as R3. It’s also generating a type 5 external LSA to flood into area 0. This is because R2 is an ABR.

<pre><code><strong>R1#show ip ospf database | begin Type-5
</strong>		Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
11.11.11.0      1.1.1.1         248         0x80000001 0x000F44 0
33.33.33.0      2.2.2.2         84          0x80000001 0x00998F 0
</code></pre>

R1 only has a type 5 external LSA in the LSDB for this prefix. This proves that our type 7 external LSA only lives within the NSSA.

That’s it! Those are all the LSA types we have for OSPF and their different functions. I can recommend to look at the OSPF LSDB a couple of times when you are doing labs.
