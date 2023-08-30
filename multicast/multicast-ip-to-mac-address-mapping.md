# Multicast IP to MAC Address Mapping

Multicast IP addresses live in the 224.0.0.0 – 239.255.255.255 range but what about MAC addresses and Ethernet frames? What do we do on layer 2 to make multicast work? Let me show you an example of a MAC address:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/03/mac-broadcast-multicast-bit.png" alt=""><figcaption></figcaption></figure>

Above, you see an example of a MAC address. In the first octet, bit 0 has been reserved for broadcast or multicast traffic. When we have unicast traffic, this bit will be set to 0. For broadcast or multicast traffic, this bit will be set to 1.

On layer 3, IANA has reserved the class D range (224.0.0.0 – 239.255.255.255) for multicast IP addresses. What about layer 2? What MAC addresses do we use for multicast traffic?

For layer 2, we also have a reserved prefix for multicast traffic. The **24-bit MAC address prefix 01-00-5E** is reserved for layer 2 multicast. Unfortunately only half of the MAC addresses in this 24-bit prefix can be used for multicast, this means we only have 23 bits of MAC address space to use for multicast. Here’s an illustration:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/03/multicast-mac-address-23-bit.png" alt=""><figcaption></figcaption></figure>

As you can see, the first three octets are 01-00-5E. This is the reserved range. This means there are 8+8+8 = 24 bits left for us to use. I just told you that only half of this 24-bit space is available to us, so only 23 bits can be used. Why can we only use 23 bits?

There’s a funny story about why we only have 23 bits left…back in the days (1990 something), Steve Deering was working on his research on IP multicast, and he wanted the IEEE to assign 16 OUIs (Organizational Unique Identifiers) to IP multicast MAC addresses. each OUI has 24 bits of address space, so 16 x 24 bits would supply enough MAC addresses to create a 1:1 relation between multicast IP address and multicast MAC address.

Each OUI costed $1000, and Steve’s manager didn’t want to pay 16 x $1000 = $16.000 just for MAC address space. As a result, Steve’s manager bought a single OUI (24 bit) and gave half of the space (23 bit) to Steve to use for his multicast research. Why does this matter? Let me show you:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/03/multicast-28-unique-bits.png" alt=""><figcaption></figcaption></figure>

Above, you see an IP address that has 32 bits. A multicast IP address also has 32 bits, but the first 4 bits are always the same (1110) because we use the 224.0.0.0 – 239.255.255.255 range. This means that each multicast IP address has **28 unique bits**.

Now if we want to map our 28 bit multicast IP address to our 23 bit MAC address, we have a problem…we miss **5 bits of mapping information:**

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/03/multicast-ip-to-mac-mapping.png" alt=""><figcaption></figcaption></figure>

This means we have to map multiple Multicast IP addresses to the same Multicast MAC address. We don’t have enough MAC addresses to give each multicast IP address its own MAC address.

We miss 5 bits of mapping information: 25 = 32. This means we will map **32 multicast IP addresses to 1 multicast MAC address**. Here’s an example:

* 224.1.1.1
* 224.129.1.1
* 225.1.1.1
* …
* …
* …
* 238.1.1.1
* 238.129.1.1
* 239.1.1.1

The multicast IP addresses above all map to the same multicast MAC address (01-00-5E-01-01-01). This can cause some problems in our networks. For example, a host that listens to the 239.1.1.1 multicast IP address will configure its network card to listen to MAC address 01-00-5E-01-01-01. If someone else is streaming to the 224.1.1.1 multicast IP address, it will also end up at our host because the MAC address is the same. The host will have to look at the IP address of the received frame to see if it’s for 239.1.1.1 and discard frames that are meant for 224.1.1.1.

Now the big question remains…what multicast IP addresses map to which multicast MAC address, and how do we calculate this? You can use a calculator, of course, but if you are studying for a Cisco exam, you don’t have this luxury. Let’s take a look at how to do this!

First, we’ll figure out which multicast MAC address maps to which 32 multicast IP addresses. You can use the following table  to calculate between decimal, hexadecimal, and binary:

| Decimal     | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   | 11   | 12   | 13   | 14   | 15   |
| ----------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Hexadecimal | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | A    | B    | C    | D    | E    | F    |
| Binary      | 0000 | 0001 | 0010 | 0011 | 0100 | 0101 | 0110 | 0111 | 1000 | 1001 | 1010 | 1011 | 1100 | 1101 | 1110 | 1111 |

We will take the following multicast MAC address and calculate what 32 multicast IP addresses map to it:

**01:00:5e:0b:01:02**

First, we have to translate this MAC address from hexadecimal to binary:

| 0    | 1    | 0    | 0    | 5    | e    | 0    | b    | 0    | 1    | 0    | 2    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 0000 | 0001 | 0000 | 0000 | 0101 | 1110 | 0000 | 1011 | 0000 | 0001 | 0000 | 0010 |

Above, you can see how I translated the hexadecimal address into binary. This is the full MAC address:

Now we will take the lowest 23 bits of this MAC address:

| 0000 0001 | 0000 0000 | 0101 1110 | 0<mark style="color:red;">000 1011</mark> | <mark style="color:red;">0000 0001</mark> | <mark style="color:red;">0000 0010</mark> |
| --------- | --------- | --------- | ----------------------------------------- | ----------------------------------------- | ----------------------------------------- |

The bits I highlighted in red are the lowest 23 bits of the MAC address.

Now we will take the class D multicast IP address range in binary:

| <mark style="color:blue;">1110</mark> <mark style="color:green;">0000</mark> | <mark style="color:green;">0</mark>000 0000 | 0000 0000 | 0000 0000 |
| ---------------------------------------------------------------------------- | ------------------------------------------- | --------- | --------- |

The digits in blue (1110) are the class D IP address in binary (224 in decimal). The green digits are the 5 bits we lose because we have to map a 28 bit unique multicast IP address to a 23 bit multicast MAC address. We will take the blue and green digits and put the red digits behind them:

| <mark style="color:blue;">1110</mark> <mark style="color:green;">0000</mark> | <mark style="color:green;">0</mark><mark style="color:red;">000 1011</mark> | <mark style="color:red;">0000 0001</mark> | <mark style="color:red;">0000 0010</mark> |
| ---------------------------------------------------------------------------- | --------------------------------------------------------------------------- | ----------------------------------------- | ----------------------------------------- |

Let’s convert this binary address into a decimal IP address:

<table data-full-width="false"><thead><tr><th>224</th><th>11</th><th>1</th><th>2</th></tr></thead><tbody><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0000</mark></td><td><mark style="color:green;">0000 1011</mark></td><td><mark style="color:red;">0000 0001</mark></td><td><mark style="color:red;">0000 0010</mark></td></tr></tbody></table>

So the complete multicast IP address is 224.11.1.2. Now we can play with the green digits to see what other multicast IP addresses map to the same MAC address:

<table><thead><tr><th width="381">Binary Multicast IP Address</th><th>Decimal Multicast IP Address</th></tr></thead><tbody><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0000</mark> <mark style="color:green;">0000 1011 0000 0001 0000 0010</mark></td><td>224.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0001 0000 1011 0000 0001 0000 0010</mark></td><td>225.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0010 0000 1011 0000 0001 0000 0010</mark></td><td>226.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0011 0000 1011 0000 0001 0000 0010</mark></td><td>227.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0100 0000 1011 0000 0001 0000 0010</mark></td><td>228.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0101 0000 1011 0000 0001 0000 0010</mark></td><td>229.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0110 0000 1011 0000 0001 0000 0010</mark></td><td>230.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0111 0000 1011 0000 0001 0000 0010</mark></td><td>231.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1000 0000 1011 0000 0001 0000 0010</mark></td><td>232.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1001 0000 1011 0000 0001 0000 0010</mark></td><td>233.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1010 0000 1011 0000 0001 0000 0010</mark></td><td>234.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1011 0000 1011 0000 0001 0000 0010</mark></td><td>235.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1100 0000 1011 0000 0001 0000 0010</mark></td><td>236.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1101 0000 1011 0000 0001 0000 0010</mark></td><td>237.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1110 0000 1011 0000 0001 0000 0010</mark></td><td>238.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1111 0000 1011 0000 0001 0000 0010</mark></td><td>239.11.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0000 1000 1011 0000 0001 0000 0010</mark></td><td>224.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0001 1000 1011 0000 0001 0000 0010</mark></td><td>225.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0010 1000 1011 0000 0001 0000 0010</mark></td><td>226.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0011 1000 1011 0000 0001 0000 0010</mark></td><td>227.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0100 1000 1011 0000 0001 0000 0010</mark></td><td>228.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0101 1000 1011 0000 0001 0000 0010</mark></td><td>229.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0110 1000 1011 0000 0001 0000 0010</mark></td><td>230.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">0111 1000 1011 0000 0001 0000 0010</mark></td><td>231.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1000 1000 1011 0000 0001 0000 0010</mark></td><td>232.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1001 1000 1011 0000 0001 0000 0010</mark></td><td>233.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1010 1000 1011 0000 0001 0000 0010</mark></td><td>234.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1011 1000 1011 0000 0001 0000 0010</mark></td><td>235.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1100 1000 1011 0000 0001 0000 0010</mark></td><td>236.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1101 1000 1011 0000 0001 0000 0010</mark></td><td>237.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1110 1000 1011 0000 0001 0000 0010</mark></td><td>238.139.1.2</td></tr><tr><td><mark style="color:blue;">1110</mark> <mark style="color:green;">1111 1000 1011 0000 0001 0000 0010</mark></td><td>239.139.1.2</td></tr></tbody></table>

There you have it. All the multicast IP addresses that map to multicast MAC address 01:00:5e:0b:01:02. Now you know how it is done in binary, you can learn a faster method to calculate these IP addresses.

With the first four green digits, the highest number we can create is 15. We start at 224.11.1.2 and end at 239.11.1.2. Those are 16 IP addresses.

The 5th green digit represents a value of 128 in the second octet. When we add 128 to 11 (the second octet of 224.11.1.2) we end up with 224.139.1.2.

The ranges we end up with are:

* 224.11.1.2 – 239.11.1.2
* 224.139.1.2 – 239.139.1.2

That’s all there is to it. Of course, we can also calculate the other way around. If we have a given multicast IP address, we can calculate what multicast MAC address it will use. I’ll use one of the IP addresses that we found above:

**226.139.1.2**

First, we have to calculate this decimal IP address into binary:

| 226      | 139                                       | 1                                         | 2                                         |
| -------- | ----------------------------------------- | ----------------------------------------- | ----------------------------------------- |
| 11100010 | 1<mark style="color:red;">000 1011</mark> | <mark style="color:red;">0000 0001</mark> | <mark style="color:red;">0000 0010</mark> |

We are only concerned with the last 23 bits of this IP address, so I made them red.

Now we need to convert these 23 bits into hexadecimal:

| 0   | b    | 0    | 1    | 0    | 2    |
| --- | ---- | ---- | ---- | ---- | ---- |
| 000 | 1011 | 0000 | 0001 | 0000 | 0010 |

We will end up with 0b:01:02. The first part of a multicast MAC address is always 01:00:5e. Let’s combine those two, and we end up with:

**01:00:5e:0b:01:02**

There you have it, our multicast MAC address for multicast IP address 226.139.1.2.\


I hope this lesson is helpful to you in calculating these multicast addresses. If you have any questions, feel free to leave a comment!
