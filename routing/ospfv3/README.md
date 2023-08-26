# OSPFv3

## OSPFv2 vs OSPFv3

OSPFv2 and OSPFv3 are very similar. OSPFv3 still establishes neighbor adjacencies, has areas, different network types, the same metrics, runs SPF, etc. There are however some differences. OSPFv2 runs on top of IPv4 and since OSPFv3 runs on IPv6, some changes had to be made.

Here are some of the differences:

* **Link-local addresses:** OSPFv3 packets are sourced from link-local IPv6 addresses.
* **Links, not networks**: OSPFv3 uses the terminology _links_ where we use _networks_ in OSPFv2.
* **New LSA types**: there are two new LSA types, and LSA type 1 and 2 have changed.
* **Interface commands**: OSPFv3 uses interface commands to enable it on the interface, we don’t use the network command anymore as OSPFv2 does.
* **OSPFv3 router ID**: OSPFv3 is unable to set its own router ID like OSPFv2 does. Instead, you have to manually configure the router ID. It is configured as a 32-bit value, same as in OSPFv2.
* **Multiple prefixes per interface**: if you have multiple IPv6 prefixes on an interface then OSPFv3 will advertise all of them.
* **Flooding scope**: OSPFv3 has a flooding scope for different LSAs.
* **Multiple instances per link**: You can run multiple OSPFv3 instances on a single link.
* **Authentication**: OSPFv3 doesn’t use plain text or MD5 authentication as OSPFv2 does. Instead, it uses IPv6’s IPSec authentication.
* **Prefixes in LSAs**: OSPFv2 shows networks in LSAs as network + subnet mask, OSPFv3 shows prefixes as prefix + prefix length.

### LSA Types

OSPFv3 has two new LSAs, and some of the LSA types have been renamed. Here is an overview of all OSPFv2 and OSPFv3 LSA types:

|  **OSPFv3**   |                       |  **OSPFv2**   |                      |
| ------------- | --------------------- | ------------- | -------------------- |
| LSA Type Code | Name                  | LSA Type Code | Name                 |
| 0x2001        | Router LSA            | 1             | Router LSA           |
| 0x2002        | Network LSA           | 2             | Network LSA          |
| 0x2003        | Inter-Area Prefix LSA | 3             | Network Summary LSA  |
| 0x2004        | Inter-Area Router LSA | 4             | ASBR Summary LSA     |
| 0x4005        | AS-External LSA       | 5             | AS-External LSA      |
| 0x2006        | Group Membership LSA  | 6             | Group Membership LSA |
| 0x2007        | Type-7 LSA            | 7             | NSSA External LSA    |
| 0x0008        | Link LSA              |               |                      |
| 0x2009        | Intra-Area Prefix LSA |               |                      |

The LSA types are still the same except type 3 is now called the Inter-Area Prefix LSA and type 4 is called the Inter-Area router LSA. The last two types, the link LSA, and intra-area prefix LSA are new to OSPFv3.

In OSPFv2, type 1 and type 2 LSAs are used for topology _and_ network information. A single LSA contains information about the topology and the networks that are used.

If you make a simple change, like changing the IP address on one of your routers then the topology itself doesn’t change. In OSPFv2, a new type 1 LSA and perhaps a type 2 LSA have to be flooded. Other routers that receive the new LSA(s) have to recalculate the SPT even though the topology did not change.

In OSPFv3, they changed this by creating a _separation_ between prefixes and the SPF tree. There is no prefix information in LSA type 1 and 2, you only find topology adjacencies in these LSAs, you don’t find any IPv6 prefixes in them. Prefixes are now advertised in type 9 LSAs and the link-local addresses that are used for next hops are advertised in type 8 LSAs. Type 8 LSAs are only flooded on the local link, type 9 LSAs are flooded within the area. The designers of OSPFv3 could have included link-local addresses in type 9 LSAs but since these are only required on the local link, it would be a waste of resources.

By separating the SPF tree and prefixes, OSPFv3 is more efficient. When the link-local address on an interface changes, the router only has to flood an updated link LSA and intra-area-prefix LSA. Since there are no changes to the topology, we don’t have to flood type 1 and 2 LSA(s). Other routers won’t have to run SPF in this case.

### Flooding Scope

In the table with LSA types above, you can see that the LSA types of OSPFv3 are hexadecimal values. The first part defines the flooding scope of the LSA:

* **0x0**: the link-local scope that is used for the Link LSA, a new LSA type for OSPFv3.
* **0x2**: the area scope, used for LSAs that are flooded throughout a single area. This is used for router, network, inter-area prefixes, inter-area router and intra-area prefix LSA types.
* **0x4**: the AS scope, used for LSAs that are flooded within the OSPFv3 routing domain, used for external LSAs.

### Headers

OSPFv2 and OSPFv3 use different headers. Here are some of the differences:

| Field            | OSPFv3   | OSPFv2            |
| ---------------- | -------- | ----------------- |
| Header Size      | 16 bytes | 24 bytes          |
| Router & Area ID | 32 bit   | 32 bit            |
| Instance ID      | Yes      | No                |
| Authentication   | IPSec    | plain text or MD5 |

The instance ID is a new field that can be used to run multiple OSPFv3 instances on a single link. OSPFv3 routers will only become neighbors if the instance ID is the same, which is 0 by default. This allows you to run OSPFv3 on a broadcast network and only form neighbor adjacencies with specific neighbors that use the same instance ID.

### Conclusion

This lesson should give you a quick overview of some of the differences between OSPFv2 and OSPFv3. In other lessons, you will learn how to configure OSPFv3.

OSPFv3 also supports IPv4, you can find a configuration example in our [OSPFv3 for IPv4 lesson](https://networklessons.com/cisco/ccnp-encor-350-401/ospfv3-for-ipv4-configuration).
