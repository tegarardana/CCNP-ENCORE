# How to troubleshoot VLANs and Trunks

In a [previous lesson](https://networklessons.com/switching/troubleshooting-interfaces) I explained some of the possible interface issues that we can encounter. Once you verified that your interface(s) are configured correctly and you are still having issues, the problem might be related to VLANs & Trunks. Let’s take a look at some common issues and how to solve them.

### VLAN assignment issues

Here is the topology:

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

H1 is unable to ping H2. There are no issues with the hosts, the problem is related to the switch. Let’s see what happens when we try a ping:

<pre><code><strong>C:>Documents and Settings>H1>ping 192.168.1.2
</strong>
Pinging 192.168.1.2 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
</code></pre>

The two computers are unable to ping each other (what a surprise!). Let’s do a quick check if there are any interface errors:

<pre><code><strong>SW1#show ip int brief
</strong>Interface           IP-Address      OK? Method Status                Protocol
FastEthernet0/1     unassigned      YES unset  up                    up      
FastEthernet0/3     unassigned      YES unset  up                    up
</code></pre>

The interfaces are looking good, no errors here. Let’s check the VLAN assignments:

<pre><code><strong>SW1#show vlan 
</strong>
VLAN Name                Status    Ports
---- -------------------------------- --------- -------------------------------
1    default             active    Fa0/1, Fa0/2, Fa0/4, Fa0/5
                                   Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                   Fa0/10, Fa0/11, Fa0/12,Fa0/13
                                   Fa0/14, Fa0/15, Fa0/16,Fa0/17
                                   Fa0/18, Fa0/19, Fa0/20,Fa0/21
                                   Fa0/22, Fa0/23, Fa0/24, Gi0/1
                                   Gi0/2
2    VLAN0002            active    Fa0/3
</code></pre>

At this moment it’s a good idea to check the VLAN information. You can use the show vlan command to quickly verify to which VLAN the interfaces belong.\
As you can see our interfaces are not in the same VLAN. Let’s fix this:

<pre><code><strong>SW1(config)#interface fa0/3
</strong><strong>SW1(config-if)#switchport access vlan 1
</strong></code></pre>

We’ll move interface Fa0/3 back to VLAN 1, both hosts are now in VLAN 1. Let’s try that ping again:

<pre><code><strong>C:>Documents and Settings>H1>ping 192.168.1.2
</strong>
Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
</code></pre>

This solves our problem!

**Lesson learned: Make sure the interface is in the correct VLAN.**

### Switchport mode issues

Time for another problem, same topology:

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

We verified that there are no interface errors, the interfaces are up and running:

<pre><code><strong>SW1#show ip interface brief 
</strong>Interface          IP-Address      OK? Method Status               Protocol
FastEthernet0/1     unassigned      YES unset  up                    up      
FastEthernet0/3     unassigned      YES unset  up                    up
</code></pre>

The interfaces don’t show any errors. Let’s check the VLAN assignments:

<pre><code><strong>SW1#show vlan 
</strong>
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/2, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13,Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17,Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21,Fa0/22
                                                Fa0/23, Fa0/24, Gi0/1, Gi0/2
10   VLAN0010                         active    Fa0/1
</code></pre>

Above you can see that FastEthernet 0/1 is in VLAN 10 but I don’t see FastEthernet 0/3 anywhere. Here are the possible causes:

* Something is wrong with the interface. We proved this wrong because it shows up/up so it seems to be active.
* The interface is not an access port but a trunk.

Let’s check the switchport information:

<pre><code><strong>SW1#show interfaces fa0/3 switchport 
</strong>Name: Fa0/3
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 10 (VLAN0010)
Trunking Native Mode VLAN: 1 (default)
</code></pre>

A quick look at the switchport information shows us what we need to know. We can confirm that interface fa0/3 is in trunk mode and the native VLAN is 1. This means that whenever H2 sends traffic and doesn’t use 802.1Q tagging that our traffic ends up in VLAN 1. Let’s turn this interface into access mode:

<pre><code><strong>SW1(config)#interface fa0/3
</strong><strong>SW1(config-if)#switchport mode access 
</strong><strong>SW1(config-if)#switchport access vlan 10
</strong></code></pre>

We’ll turn FastEthernet 0/3 into access mode and make sure it’s in VLAN 10. Let’s verify this:

<pre><code><strong>SW1#show vlan id 10
</strong>
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
10   VLAN0010                         active    Fa0/1, Fa0/3
</code></pre>

Both interfaces are now active in VLAN 10. Checking the operational mode is also a good idea:

<pre><code><strong>SW1#show interfaces fa0/3 switchport | include Operational Mode 
</strong>Operational Mode: static access
</code></pre>

It now shows up as access mode. Let’s try that ping again:

<pre><code><strong>C:>Documents and Settings>H1>ping 192.168.1.2
</strong>
Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
</code></pre>

Now I can send a ping from H1 to H2…problem solved!

**Lesson learned: Make sure the interface is in the correct switchport mode (access or trunk mode).**

### VACL (VLAN Access-List) issues

Same two computers, same switch, different problem:

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

This scenario is a bit more interesting though. The computers are unable to ping each other so let’s walk through our list of “possible” errors:

<pre><code><strong>SW1#show ip interface brief 
</strong>Interface          IP-Address      OK? Method Status               Protocol
FastEthernet0/1     unassigned      YES unset  up                    up      
FastEthernet0/3     unassigned      YES unset  up                    up
</code></pre>

The interfaces are looking good, up/up is what we like to see. Let’s check VLAN assignments:

<pre><code><strong>SW1#show vlan
</strong>
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active   Fa0/2, Fa0/4, Fa0/5, Fa0/6
                                               Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                               Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                               Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                               Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                               Fa0/23, Fa0/24, Gi0/1, Gi0/2
10   VLAN0010                         active   Fa0/1, Fa0/3
</code></pre>

Both interfaces are in VLAN 10 so this looks ok to me. Port-security perhaps?

<pre><code><strong>SW1#show port-security 
</strong>Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                (Count)       (Count)          (Count)
---------------------------------------------------------------------------
---------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 6144
</code></pre>

Just to be sure…there’s no port security. This is an interesting situation. The interfaces are up/up, we are in the same VLAN and there’s no port security. Anything else that could block traffic? You bet there is…

<pre><code><strong>SW1#show vlan filter 
</strong>VLAN Map BLOCKSTUFF is filtering VLANs:
  10
</code></pre>

This might not be something you think about immediately but we can use VACLs (VLAN access-list) to permit or deny traffic within the VLAN. If you are troubleshooting switches then this is something you’ll have to look at if everything else seems fine. In this case there’s a VACL attached to VLAN 10, let’s inspect it:

<pre><code><strong>SW1#show vlan access-map 
</strong>Vlan access-map "BLOCKSTUFF"  10
  Match clauses:
    ip  address: 1
  Action:
    drop
Vlan access-map "BLOCKSTUFF"  20
  Match clauses:
  Action:
    forward
</code></pre>

There are two sequence numbers…10 and 20. Sequence number 10 matches on access-list 1 and the action is to drop traffic. Let’s take a look what this access-list 1 is about:

<pre><code><strong>SW1#show access-lists 
</strong>Standard IP access list 1
    10 permit 192.168.1.0, wildcard bits 0.0.0.255
</code></pre>

Don’t be confused because of the permit statement here. Using a permit statement in the access-list means that it will “match” on 192.168.1.0 /24. Our two computers are using IP addresses from this range. If it matches this access-list then the VLAN access-map will drop the traffic. Let’s change it:

<pre><code><strong>SW1(config)#vlan access-map BLOCKSTUFF 10
</strong><strong>SW1(config-access-map)#action forward
</strong></code></pre>

Let’s change the action to forward and see if it solves our problem:

<pre><code><strong>C:>Documents and Settings>H1>ping 192.168.1.2
</strong>
Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
</code></pre>

There we go, our ping is now working.

**Lesson learned: If everything else seems to be ok, make sure there’s no VACL!**

### Trunking issues

So far we checked VLAN issues on a single switch. When everything checks out with your VLANs and the traffic has to pass one or more switches then you might want to check the trunks. Let’s take a look at this topology:

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Above we have two switches, behind each switch is a host in VLAN 10. Unfortunately the hosts are unable to reach each other. Where do we start?

Let’s check the interface on SW1 that connects to the host first:

<pre><code><strong>SW1#show interfaces fa0/1
</strong>FastEthernet0/1 is up, line protocol is up (connected)
</code></pre>

<pre><code><strong>SW1#show interfaces fa0/1 switchport
</strong>Name: Fa0/1
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: On
Access Mode VLAN: 10 (VLAN0010)
</code></pre>

<pre><code><strong>SW1#show port-security interface fa0/1
</strong>Port Security : Disabled
</code></pre>

The interface is up and running, it’s a switchport and assigned to VLAN 10. This is looking good so far. Port security is not enabled so we don’t have to worry about it. Let’s do the same check on SW2:

<pre><code><strong>SW2#show interfaces fa0/2
</strong>FastEthernet0/2 is up, line protocol is up (connected)

<strong>SW2#show interfaces fa0/2 switchport
</strong>Name: Fa0/2
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: On
Access Mode VLAN: 10 (VLAN0010)
</code></pre>

<pre><code><strong>SW2#show port-security interface fa0/2
</strong>Port Security : Disabled
</code></pre>

I’ll check the same things on SW2. The interface is operational and it has been assigned to VLAN 10.

At this moment we know that the interfaces to the computers are configured correctly. At this moment you could do two things:

* Connect another computer to SW1 and assign it to VLAN 10. See if you can communicate between computers in VLAN 10 when they are connected to the same switch. Do the same on SW2.
* Check the interface between SW1 and SW2.

I’m going to focus on the interface between SW1 and SW2 because there’s plenty that could go wrong there! Let’s take a look:

<pre><code><strong>SW1#show interfaces fa0/15
</strong>FastEthernet0/15 is up, line protocol is up (connected)
</code></pre>

<pre><code><strong>SW2#show interfaces fa0/15
</strong>FastEthernet0/15 is up, line protocol is up (connected)
</code></pre>

The interfaces are showing no issues, time to check the switchport information:

<pre><code><strong>SW1#show interfaces fa0/15 switchport 
</strong>Name: Fa0/15
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: isl
Operational Trunking Encapsulation: isl
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
</code></pre>

SW1 is in trunk mode and using ISL encapsulation. What about SW2?

<pre><code><strong>SW2#show interfaces fa0/15 switchport 
</strong>Name: Fa0/15
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)Trunking Native Mode VLAN: 1 (default)
</code></pre>

SW2 is also in trunk mode but using 802.1Q encapsulation. Be aware that (depending on the switch model) the default administrative mode might be dynamic auto. Two interfaces that are both running in dynamic auto mode will become access ports. It’s best to change the interface to trunk mode by yourself. In our case both interfaces are trunking so that’s good but we have an encapsulation protocol mismatch. Let’s fix that:

<pre><code><strong>SW1(config)#interface fa0/15
</strong><strong>SW1(config-if)#switchport trunk encapsulation dot1q
</strong></code></pre>

We’ll change the encapsulation type so both switches are using 802.1Q. Is that it?

<pre><code><strong>C:>Documents and Settings>H1>ping 192.168.1.2
</strong>
Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
</code></pre>

Problem solved! Our ping is working.

**Lesson learned: Make sure you use the same encapsulation protocol when configuring trunks.**

### Trunk allowed VLANs issue

Let’s look at another trunk issue, same topology:

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Assume we checked and verified that the following items are causing no issues:

* Interfaces (speed/duplex)
* Port-security.
* Switchport configuration (VLAN assignment, interface configured in access mode).

Our poor hosts are unable to reach each other:

<pre><code><strong>C:>Documents and Settings>H1>ping 192.168.1.2
</strong>
Pinging 192.168.1.2 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
</code></pre>

Let’s check the trunk in between the switches:

<pre><code><strong>SW1#show interfaces fa0/15 switchport 
</strong>Name: Fa0/15
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
</code></pre>

<pre><code><strong>SW2#show interfaces fa0/15 switchport 
</strong>Name: Fa0/15
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
</code></pre>

I’ll verify that both interfaces are in trunk mode and that we are using the same encapsulation protocol (802.1Q). No problems here. Anything else that can go wrong with this trunk link? You bet!

<pre><code><strong>SW1#show interfaces fa0/15 trunk 
</strong>
Port        Mode             Encapsulation  Status        Native vlan
Fa0/15      on               802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/15      20
</code></pre>

<pre><code><strong>SW2#show interfaces fa0/15 trunk 
</strong>
Port        Mode             Encapsulation  Status        Native vlan
Fa0/15      on               802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/15      20
</code></pre>

The trunk might be operational but this doesn’t mean that all VLANs are allowed over the trunk link. In the example above you can see that only VLAN 20 is allowed. Let’s fix that:

<pre><code><strong>SW1(config)#interface fa0/15
</strong><strong>SW1(config-if)#switchport trunk allowed vlan all
</strong></code></pre>

<pre><code><strong>SW2(config)#interface fa0/15
</strong><strong>SW2(config-if)#switchport trunk allowed vlan all
</strong></code></pre>

Let’s allow all VLANs to pass the trunk. Let’s try the ping again:

<pre><code><strong>C:>Documents and Settings>H1>ping 192.168.1.2
</strong>
Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
</code></pre>

VLAN 10 is now able to pass the trunk link between the two switches. As a result I can ping between the computers….another problem bites the dust!

**Lesson learned: Always check if a trunk allows all VLANs or not.**

That’s all I have for now, hopefully this has been useful to find and fix VLAN and trunk related issues on your switches. If you have any questions, feel free to leave a comment!
