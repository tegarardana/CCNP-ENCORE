# Troubleshooting Etherchannel

Most of the issues with Etherchannels (Link Aggregation) are because of misconfiguration. Keep in mind that the configuration of all physical interfaces has to match. In this lesson we’ll take a look at some common issues with Etherchannels.

This is the topology I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/switch-1-2-etherchannel.png" alt=""><figcaption></figcaption></figure>

The idea is to bundle FastEthernet 0/13 and 0/14 into an Etherchannel but this is not working. Let’s look at the first issue…

### Wrong Etherchannel Mode

Let’s look at our first issue, the Etherchannel is not working. First we’ll check if the interfaces are up and running:

<pre><code><strong>SW1#show interfaces fa0/13 | include line protocol
</strong>FastEthernet0/13 is up, line protocol is up (connected)
</code></pre>

<pre><code><strong>SW1#show interfaces fa0/14 | include line protocol
</strong>FastEthernet0/14 is up, line protocol is up (connected)
</code></pre>

<pre><code><strong>SW2#show interfaces fa0/13 | include line protocol
</strong>FastEthernet0/13 is up, line protocol is up (connected)
</code></pre>

<pre><code><strong>SW2#show interfaces fa0/14 | include line protocol
</strong>FastEthernet0/14 is up, line protocol is up (connected)
</code></pre>

All interfaces are operational. Let’s check if we have a port-channel interface:

<pre><code><strong>SW1#show ip int brief | include Port
</strong>Port-channel1          unassigned      YES unset  down                  down
</code></pre>

<pre><code><strong>SW2#show ip int brief | include Port
</strong>Port-channel1          unassigned      YES unset  down                  down 
</code></pre>

We can verify that a port-channel interface has been created but it is down. Let’s check its details:

<pre><code><strong>SW1#show etherchannel summary | begin Group
</strong>Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SD)         LACP      Fa0/13(I)   Fa0/14(I)
</code></pre>

<pre><code><strong>SW2#show etherchannel summary | begin Group
</strong>Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SD)         PAgP      Fa0/13(I)   Fa0/14(I)
</code></pre>

Here’s a nice command to verify your etherchannel. Use the **`show etherchannel summary`** to see your port-channels. We can see that SW1 has been configured for LACP and SW2 for PAgP, this is never going to work. Let’s take a closer look:

<pre><code><strong>SW1#show etherchannel 1 detail 
</strong>Group state = L2 
Ports: 2   Maxports = 16
Port-channels: 1 Max Port-channels = 16
Protocol:   LACP
Minimum Links: 0
Ports in the group:
-------------------
Port: Fa0/13
------------

Port state    = Up Sngl-port-Bndl Mstr Not-in-Bndl 
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = null        GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Fa0/13    SA      indep     32768         0x1       0x1     0xF         0x7D  

Age of the port in the current state: 0d:00h:11m:59s
          
Port: Fa0/14
------------
          
Port state    = Up Sngl-port-Bndl Mstr Not-in-Bndl 
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = null        GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP
          
Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.
          
Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Fa0/14    SA      indep     32768         0x1       0x1     0x10        0x7D  
          
Age of the port in the current state: 0d:00h:12m:38s
          
Port-channels in the group: 
---------------------------
          
Port-channel: Po1    (Primary Aggregator)
------------
          
Age of the Port-channel   = 0d:00h:12m:55s
Logical slot/port   = 2/1          Number of ports = 0
HotStandBy port = null 
Port state          = Port-channel Ag-Not-Inuse 
Protocol            =   LACP
Port security       = Disabled
</code></pre>

The best command to use is **show etherchannel detail**. This gives you a lot of information but I’m particularly interested in seeing if LACP is configured for **passive** or **active** mode. Interfaces in active mode will “actively” try to form an etherchannel. Interfaces in passive mode will only respond to LACP requests. Let’s look at SW2:

<pre><code><strong>SW2#show etherchannel 1 detail
</strong>Group state = L2 
Ports: 2   Maxports = 8
Port-channels: 1 Max Port-channels = 1
Protocol:   PAgP
Minimum Links: 0
Ports in the group:
-------------------
Port: Fa0/13
------------

Port state    = Up Sngl-port-Bndl Mstr Not-in-Bndl 
Channel group = 1           Mode = Desirable-Sl    Gcchange = 0
Port-channel  = null        GC   = 0x00010001      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/13    	U4/S4   H	30s	 0        128        Any      10013
          
Age of the port in the current state: 0d:00h:15m:25s
          
Port: Fa0/14
------------
          
Port state    = Up Sngl-port-Bndl Mstr Not-in-Bndl 
Channel group = 1           Mode = Desirable-Sl    Gcchange = 0
Port-channel  = null        GC   = 0x00010001      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP
          
Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.
          
Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/14    	U4/S4   H	30s	 0        128        Any      10014
          
Age of the port in the current state: 0d:00h:15m:28s
          
Port-channels in the group: 
---------------------------
          
Port-channel: Po1
------------
          
Age of the Port-channel   = 0d:00h:15m:51s
Logical slot/port   = 2/1          Number of ports = 0
GC                  = 0x00000000      HotStandBy port = null
Port state          = Port-channel Ag-Not-Inuse 
Protocol            =   PAgP
Port security       = Disabled
</code></pre>

Here’s the output of show etherchannel detail of SW2. We can see that it has been configured for PAgP and the interfaces are configured for **desirable** mode. If they would have been configured for auto mode we would see the **A flag**. Let’s re-configure SW2 so that is uses LACP:

<pre><code><strong>SW2(config)#no interface po1
</strong><strong>SW2(config)#interface fa0/13
</strong><strong>SW2(config-if)#channel-group 1 mode passive
</strong><strong>SW2(config-if)#exit
</strong><strong>SW2(config)#interface fa0/14
</strong><strong>SW2(config-if)#channel-group 1 mode passive
</strong></code></pre>

Let’s get rid of the port-channel interface first, if we don’t do this you’ll see an error when you try to change the channel-group mode on the interfaces. After a few seconds you’ll see the port-channel interface appear:

```
SW1# %LINK-3-UPDOWN: Interface Port-channel1, changed state to up
```

```
SW2# %LINK-3-UPDOWN: Interface Port-channel1, changed state to up
```

After changing the configuration we see the port-channel1 going up. Problem solved!

**Lesson learned: Make sure you use the same EtherChannel mode (PaGP or LACP) on both sides.**

### PaGP / LACP Negotiation not working

Same topology, different problem:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/switch-1-2-etherchannel.png" alt=""><figcaption></figcaption></figure>

Once again the Etherchannel is down:

<pre><code><strong>SW1#show ip interface brief | include Port
</strong>Port-channel1          unassigned      YES unset  down                  down
</code></pre>

<pre><code><strong>SW2#show ip interface brief | include Port
</strong>Port-channel1          unassigned      YES unset  down                  down
</code></pre>

We can verify that the port-channel interface exists but it’s down on both sides. Let’s check the Etherchannel configuration:

<pre><code><strong>SW1#show etherchannel 1 summary | begin Group
</strong>Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SD)         PAgP      Fa0/13(D)   Fa0/14(D)   


<strong>SW2#show etherchannel 1 summary | begin Group
</strong>Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SD)         PAgP      Fa0/13(D)   Fa0/14(D)
</code></pre>

We can also see that interface FastEthernet 0/13 and 0/14 have both been added to the port-channel interface and that we use PAgP.

<pre><code><strong>SW1#show ip interface brief | begin FastEthernet0/13
</strong>FastEthernet0/13       unassigned      YES unset  up                    up      
FastEthernet0/14       unassigned      YES unset  up                    up 
</code></pre>

```
SW2#show ip interface brief | begin FastEthernet0/13
FastEthernet0/13       unassigned      YES unset  up                    up      
FastEthernet0/14       unassigned      YES unset  up                    up 
```

The FastEthernet interfaces are looking good so we know this is not the issue. Let’s dive deeper into the etherchannel configuration.

<pre><code><strong>SW1#show etherchannel 1 port
</strong>Ports in the group:
-------------------
Port: Fa0/13
------------

Port state    = Up Sngl-port-Bndl Mstr Not-in-Bndl 
Channel group = 1           Mode = Automatic-Sl    Gcchange = 0
Port-channel  = null        GC   = 0x00010001      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/13    A	U2/S4   	1s	 0        128        Any      10013

Age of the port in the current state: 0d:00h:11m:38s

Port: Fa0/14
------------

Port state    = Up Sngl-port-Bndl Mstr Not-in-Bndl 
Channel group = 1           Mode = Automatic-Sl    Gcchange = 0
Port-channel  = null        GC   = 0x00010001      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/14    A	U2/S4   	1s	 0        128        Any      10014

Age of the port in the current state: 0d:00h:11m:41s
</code></pre>

We can see that FastEthernet 0/13 and 0/14 on SW1 are both configured for PAgP Auto mode (because of the “A” flag). What about SW2?

<pre><code><strong>SW2#show etherchannel 1 port
</strong>		Ports in the group:
		-------------------
Port: Fa0/13
------------

Port state    = Up Sngl-port-Bndl Mstr Not-in-Bndl 
Channel group = 1           Mode = Automatic-Sl    Gcchange = 0
Port-channel  = null        GC   = 0x00010001      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/13    A	U2/S4   	1s	 0        128        Any      10013

Age of the port in the current state: 0d:00h:13m:13s

Port: Fa0/14
------------

Port state    = Up Sngl-port-Bndl Mstr Not-in-Bndl 
Channel group = 1           Mode = Automatic-Sl    Gcchange = 0
Port-channel  = null        GC   = 0x00010001      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/14    A	U2/S4   	1s	 0        128        Any      10014

Age of the port in the current state: 0d:00h:13m:13s
</code></pre>

FastEthernet 0/13 and 0/14 on SW2 are also configured for PAgP auto. This is never going to work because both switches are now waiting passively for PAgP messages. Let’s change the SW2 interfaces to desirable:

<pre><code><strong>SW2(config)#interface fa0/13
</strong><strong>SW2(config-if)#channel-group 1 mode desirable 
</strong><strong>SW2(config-if)#interface fa0/14
</strong><strong>SW2(config-if)#channel-group 1 mode desirable
</strong></code></pre>

SW2 now actively sends PAgP messages to SW1 and as a result, the port-channel negotiation will complete:

```
SW1# %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
```

```
SW2# %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
```

The etherchannel is now working…problem solved!

**Lesson learned: When using PAgP make sure at least one of the switches is using desirable mode or in case of LACP make sure one switch is in active mode.**

### Interface Configuration Mismatch

I have one more scenario for you, same topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/switch-1-2-etherchannel.png" alt=""><figcaption></figcaption></figure>

An etherchannel is configured between SW1 and SW2 but the customer is complaining that the link is slow…what could possibly be wrong?

<pre><code><strong>SW1#show ip int brief | include Port
</strong>Port-channel1          unassigned      YES unset  up                    up  
</code></pre>

<pre><code><strong>SW2#show ip int brief | include Port
</strong>Port-channel1          unassigned      YES unset  up                    up 
</code></pre>

A quick check tells us that the port-channel interface is operational. Let’s take a closer look though:

<pre><code><strong>SW1#show etherchannel 1 detail 
</strong>Group state = L2 
Ports: 2   Maxports = 8
Port-channels: 1 Max Port-channels = 1
Protocol:   PAgP
Minimum Links: 0
		Ports in the group:
		-------------------
Port: Fa0/13
------------

Port state    = Up Cnt-bndl Suspend Not-in-Bndl 
Channel group = 1           Mode = Automatic-Sl    Gcchange = 0
Port-channel  = null        GC   = 0x00000000      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/13    dA	U1/S1   	1s	 0        128        Any      0

Age of the port in the current state: 0d:01h:10m:37s

Probable reason: speed of Fa0/13 is 100M, Fa0/14 is 10M
Port: Fa0/14
------------

Port state    = Up Mstr In-Bndl 
Channel group = 1           Mode = Automatic-Sl    Gcchange = 0
Port-channel  = Po1         GC   = 0x00010001      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Fa0/14    SAC	U6/S7   HQ	30s	 1        128        Any      5001

Partner's information:

          Partner              Partner          Partner         Partner Group
Port      Name                 Device ID        Port       Age  Flags   Cap.
Fa0/14    SW2                  0019.569d.5700	Fa0/14      15s SC	10001 

Age of the port in the current state: 0d:00h:04m:29s

Port-channels in the group: 
---------------------------

Port-channel: Po1
------------

Age of the Port-channel   = 0d:01h:30m:23s
Logical slot/port   = 2/1          Number of ports = 1
GC                  = 0x00010001      HotStandBy port = null
Port state          = Port-channel Ag-Inuse 
Protocol            =   PAgP
Port security       = Disabled

Ports in the Port-channel: 

Index   Load   Port     EC state        No of bits
------+------+------+------------------+-----------
  0     00     Fa0/14   Automatic-Sl       0

Time since last port bundled:    0d:00h:04m:31s    Fa0/14
Time since last port Un-bundled: 0d:00h:08m:12s    Fa0/14
</code></pre>

The show etherchannel detail command produces a lot of output but it does tell us what’s going on. You can see that interface FastEthernet 0/13 and 0/14 both have been configured for this port-channel but the switch was unable to bundle them because FastEthernet 0/14 is configured for 10Mbit. You can see this in the probable reason that I highlighted. SW2 tells us the same thing:

<pre><code><strong>SW2#show etherchannel 1 detail | include reason
</strong>Probable reason: speed of Fa0/13 is 100M, Fa0/14 is 10M
</code></pre>

This is a good reason to use one of the operators for show command. I’m only interested in seeing the probably reason that the “show etherchannel detail” command produces. Let’s change the bandwidth for interface Fa0/14 on both switches:

<pre><code><strong>SW1(config)#interface fa0/14
</strong><strong>SW1(config-if)#speed auto
</strong></code></pre>

<pre><code><strong>SW2(config)#interface fa0/14
</strong><strong>SW2(config-if)#speed auto
</strong></code></pre>

Let’s change the speed to auto. We need to make sure that FastEthernet 0/13 and 0/14 both have the same configuration. It’s not a bad idea to do a show run to check if they have the same commands applied to them. Once we fix this you will see the Etherchannel going down and come back up:

```
SW1# %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to down
%LINK-3-UPDOWN: Interface Port-channel1, changed state to down
%LINK-3-UPDOWN: Interface Port-channel1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
```

```
SW2# %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to down
%LINK-3-UPDOWN: Interface Port-channel1, changed state to down
%LINK-3-UPDOWN: Interface Port-channel1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
```

Now let’s see if the two interfaces have been bundles:

<pre><code><strong>SW1#show etherchannel 1 summary | begin Group
</strong>Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Fa0/13(P)   Fa0/14(P)
</code></pre>

<pre><code><strong>SW2#show etherchannel 1 summary | begin Group
</strong>Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Fa0/13(P)   Fa0/14(P)
</code></pre>

Now we see that both interfaces have been added to the port-channel…problem solved!

**Lesson learned: Make sure all interfaces that will be added to the port-channel have the exact same configuration!**

These are all the Etherchannels I wanted to show you. The most common issue is that the configuration on the physical interfaces doesn’t match. Also, make sure that if you are using LACP or PAgP that one of the switches is actively negotiating. If you have any questions, feel free to leave a comment.
