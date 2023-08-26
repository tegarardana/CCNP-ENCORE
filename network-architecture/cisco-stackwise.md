# Cisco Stackwise

Cisco’s access layer switches used to be all separate physical switches where we use Ethernet cables for connectivity between the switches. Cisco Stackwise changed this, it allows us to turn multiple physical switches **into a single logical switch**.

Switches that support Stackwise use a special stacking cable to connect the switches to each other. Each switch has two stacking connectors that are used to “daisy-chain” (loop) the switches together. Each switch is connected to the one below it and the bottom switch will be connected to the one on top.

The Stackwise cable is like an extension of the switching fabric of the switches. When an Ethernet frame has to be moved from one physical switch to another, the Stackwise “loop” is used. The advantage of using a cabled loop is that you can remove one switch from the stack, the loop will be broken but the stack will keep working.

One switch in the stack becomes the **master** that does all “management tasks” for the stack. All other switches are **members**. If the master fails, another member will become the new master. To select a master, Stackwise uses an election process that checks for the following criteria (in order of importance):

1. **User priority**: we can configure a priority to decide which switch becomes the master.
2. **Default Configuration**: A switch that already has a configuration will take precedence over switches with no configuration.
3. **Hardware/software priority**: The switch with the most extensive feature set has a higher priority than another switch (for example: IP Services vs IP base).
4. **Uptime**: The switch with the longest uptime.
5. **MAC address**: The switch with the lowest MAC address.

It makes sense to choose the master ourselves so normally we use user priority to configure the master.

Once the stack has been created, the configuration of the switches is the same as if it were one single switch…they share the same management IP address, hostname, etc.

{% hint style="info" %}
The Cisco Catalyst 4500 and 6500 also support something similar to Stackwise, it’s called [VSS (Virtual Switching System) and I covered it in this lesson](https://networklessons.com/cisco/ccnp-encor-350-401/cisco-6500-vss-configuration-example).
{% endhint %}

Here’s a picture of the stacking connectors, this is the rear of a Cisco Catalyst 3750 switch:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/01/cisco-3750-rear.jpg" alt=""><figcaption></figcaption></figure>

You can see the two connectors on the left side…Stack1 and Stack 2. Here’s what the Stackwise cable looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/01/cisco-3750-stackwise-cable.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/01/cisco-3750-stackwise-connector.jpg" alt=""><figcaption></figcaption></figure>

Now you have an idea what Stackwise is about, let’s take a look at the configuration.

## Configuration

I will use two Cisco Catalyst 3750 switches to configure Stackwise. Make sure they are all powered off and then cable them like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/01/cisco-3750-stack-cables.png" alt=""><figcaption></figcaption></figure>

First I will start the switch that I want to become the Master, i’ll use the one on top for this. Once you start it with the cables connected, you’ll see some Stackwise information during boot:

```
SM: Detected stack cables at PORT1 PORT2

Waiting for Stack Master Election...
SM: Waiting for other switches in stack to boot...
##################################################
SM: All possible switches in stack are booted up

Election Complete
Switch 1 booting as Master
Waiting for Port download...Complete

%STACKMGR-4-SWITCH_ADDED: Switch 1 has been ADDED to the stack
%STACKMGR-5-SWITCH_READY: Switch 1 is READY
%STACKMGR-4-STACK_LINK_CHANGE: Stack Port 1 Switch 1 has changed to state DOWN
%STACKMGR-4-STACK_LINK_CHANGE: Stack Port 2 Switch 1 has changed to state DOWN
%STACKMGR-5-MASTER_READY: Master Switch 1 is READY
```

Our switch detected the stacking cables and does an election to see who will become the master. By default each switch will think that it’s switch number 1 and the master. I’ll change the user priority to make sure that this switch will always be selected as the master:

<pre><code><strong>SW1#switch 1 priority 15
</strong>Changing the Switch Priority of Switch Number 1 to 15
Do you want to continue?[confirm]
New Priority has been set successfully
</code></pre>

To make sure the new user priority will be used, we’ll save our configuration and reboot the switch:

<pre><code><strong>SW1#copy running-config startup-config
</strong><strong>SW1#reload
</strong></code></pre>

Once SW1 is back up and running, we’ll power on the second switch. This is what you will see on the console:

```
SW2#
SM: Detected stack cables at PORT1 PORT2

Waiting for Stack Master Election...

Election Complete
Switch 2 booting as Member, Switch 1 elected Master
HCOMP: Compatibility check PASSED 
Waiting for feature sync....
Waiting for Port download...Complete
Stack Master is ready
```

You can see that the election was succesful, the second switch has become the member. This is looking good so let’s verify our work.

### Verification

There are a number of useful show commands to verify that Stackwise is working:

<pre><code><strong>SW1#show switch 
</strong>Switch/Stack Mac Address : 0011.214e.d180
                                           H/W   Current
Switch#  Role   Mac Address     Priority Version  State 
----------------------------------------------------------
*1       Master 0011.214e.d180     15     0       Ready               
 2       Member 0016.c762.6c80     1      0       Ready
</code></pre>

The **show switch** command tells us that we have a master and member, you can also see the user priority and the switch numbers. We can also verify if both stack connectors are used or not:

<pre><code><strong>SW1#show switch stack-ports 
</strong>  Switch #    Port 1       Port 2 
  --------    ------       ------ 
    1           Ok          Ok  
    2           Ok          Ok
</code></pre>

Both switches have two stack ports that are up and running. We can also check the bandwidth that Stackwise offers:

<pre><code><strong>SW1#show switch stack-ring speed 
</strong>
Stack Ring Speed        : 32G
Stack Ring Configuration: Full
Stack Ring Protocol     : StackWise
</code></pre>

This tells us the ring speed is 32 Gbit and the configuration is “Full”. This means that we have a daisy-chained loop, if you only use a single Stackwise cable then this command will show only 16 Gbit for the ring speed and the ring configuration will be “Half”.

Our two switches have been combined into a single logical switch, here’s what the interface numbers now look like:

<pre><code><strong>SW1#show ip interface brief | include Fast
</strong>FastEthernet1/0/1      unassigned      YES unset  down                  down    
FastEthernet1/0/2      unassigned      YES unset  down                  down    
FastEthernet1/0/3      unassigned      YES unset  down                  down    
FastEthernet1/0/4      unassigned      YES unset  down                  down    
[output omitted]    
FastEthernet2/0/1      unassigned      YES unset  down                  down    
FastEthernet2/0/2      unassigned      YES unset  down                  down    
FastEthernet2/0/3      unassigned      YES unset  down                  down    
FastEthernet2/0/4      unassigned      YES unset  down                  down    
[output omitted]
</code></pre>

The interfaces on the first switch use the FastEthernet1/0/X format and the interfaces on the second switch use the FastEthernet2/0/X format.

All other devices that you connect to our Stack will only see one device. Let’s verify this by connecting another switch to the topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/01/stackwise-switches-redundant-topology.png" alt=""><figcaption></figcaption></figure>

I will connect a switch to the Stack, one interface to the master switch and the other one to the member. Here’s what CDP looks like:

<pre><code><strong>SW3#show cdp neighbors 
</strong>Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone

Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
SW1              Fas 0/21          129           S I      WS-C3750- Fas 2/0/21
SW1              Fas 0/20          150           S I      WS-C3750- Fas 1/0/20
</code></pre>

It only sees one device, our logical switch stack called SW1.

That’s all I have for now, I hope this has been helpful to understand Stackwise. If you have any questions, feel free to leave a comment.
