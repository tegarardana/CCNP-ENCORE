# How to Configure Trunks

Trunks are required to carry VLAN traffic from one switch to another. This lesson will demonstrate how to configure a trunk between Cisco Catalyst switches. Let me show you the topology that we’ll use:

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Above, you see a topology with a computer connected to each switch. We’ll put the computers in the same VLAN and create a trunk between the two switches.

Let’s start by creating a VLAN:

<pre><code><strong>SW1(config)#vlan 50
</strong><strong>SW1(config-vlan)#name Computers
</strong><strong>SW1(config-vlan)#exit
</strong></code></pre>

<pre><code><strong>SW2(config)#vlan 50
</strong><strong>SW2(config-vlan)#name Computers
</strong><strong>SW2(config-vlan)#exit
</strong></code></pre>

And let’s put the interfaces connected to the computers in the correct VLAN:

<pre><code><strong>SW1(config)#interface fa0/1
</strong><strong>SW1(config-if)#switchport access vlan 50
</strong></code></pre>

<pre><code><strong>SW2(config)#interface fa0/2
</strong><strong>SW2(config-if)#switchport access vlan 50
</strong></code></pre>

The next step is to create a trunk between the two switches. Technically the interfaces between the two switches can also be in access mode right now because I only have a single VLAN.

<pre><code><strong>SW1(config)#interface fa0/14
</strong><strong>SW1(config-if)#switchport mode trunk
</strong>Command rejected: An interface whose trunk encapsulation is "Auto" can not be configured to "trunk" mode.
</code></pre>

<pre><code><strong>SW2(config)#interface fa0/14
</strong><strong>SW2(config-if)#switchport mode trunk
</strong>Command rejected: An interface whose trunk encapsulation is "Auto" can not be configured to "trunk" mode.
</code></pre>

I try to change the interface to trunk mode with the `switchport` mode trunk command. Depending on the switch model, you might see the same error as me. If we want to change the interface to trunk mode, we need to change the trunk encapsulation type. Let’s see what options we have:

```
SW1(config-if)#switchport trunk encapsulation ?
  dot1q      Interface uses only 802.1q trunking encapsulation when trunking
  isl        Interface uses only ISL trunking encapsulation when trunking
  negotiate  Device will negotiate trunking encapsulation with peer on interface
```

This is where you can choose between 802.1Q or ISL encapsulation. By default, our switch will negotiate about the trunk encapsulation type.

<pre><code><strong>SW1(config-if)#switchport trunk encapsulation dot1q
</strong></code></pre>

```
SW2(config-if)#switchport trunk encapsulation dot1q
```

Let‟s change it to 802.1Q by using the **`switchport trunk encapsulation`** command.

<pre><code><strong>SW1#show interfaces fa0/14 switchport
</strong>Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic auto 
Operational Mode: static access 
<strong>Administrative Trunking Encapsulation: dot1q
</strong></code></pre>

<pre><code><strong>SW2#show interfaces fa0/14 switchport
</strong>Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic auto 
Operational Mode: static access 
<strong>Administrative Trunking Encapsulation: dot1q
</strong></code></pre>

As you can see the trunk encapsulation is now 802.1Q.

<pre><code><strong>SW1(config)#interface fa0/14
</strong><strong>SW1(config-if)#switchport mode trunk
</strong></code></pre>

<pre><code><strong>SW2(config)#interface fa0/14
</strong><strong>SW2(config-if)#switchport mode trunk
</strong></code></pre>

Now I can successfully change the `switchport` mode to `trunk`.

<pre><code><strong>SW1#show interfaces fa0/14 switchport
</strong>Name: Fa0/14
Switchport: Enabled Administrative Mode: trunk Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
<strong>Operational Trunking Encapsulation: dot1q
</strong></code></pre>

<pre><code><strong>SW2#show interfaces fa0/14 switchport
</strong>Name: Fa0/14
Switchport: Enabled Administrative Mode: trunk Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
<strong>Operational Trunking Encapsulation: dot1q
</strong></code></pre>

We can confirm we have a trunk because the operational mode is “dot1q”.

Let’s try if H1 and H2 can reach each other:

<pre><code><strong>C:\Documents and Settings\H1>ping 192.168.1.2
</strong>
Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128
Reply from 192.168.1.2: bytes=32 time&#x3C;1ms TTL=128

Ping statistics for 192.168.1.2:
<strong>Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
</strong>Approximate round trip times in milli-seconds:
Minimum = 0ms, Maximum = 0ms, Average = 0ms
</code></pre>

Excellent! H1 and H2 can reach each other! Does this mean we are\
done? Not quite yet…there is more I want to show to you:

```
SW2#show vlan
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/3, Fa0/4, Fa0/5
                                                Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/15, Fa0/22, Fa0/23, Fa0/24
                                                Gi0/1, Gi0/2
50   Computers                        active    Fa0/2
```

First of all, if we use the show vlan command, we don’t see the Fa0/14 interface. This is completely normal because the show vlan command only shows interfaces in access mode and no trunk interfaces.

```
SW2#show interface fa0/14 trunk 
Port        Mode             Encapsulation  Status        Native vlan
Fa0/14      on               802.1q         trunking      1
Port        Vlans allowed on trunk
Fa0/14      1-4094
Port        Vlans allowed and active in management domain
Fa0/14      1,50
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/14      50
```

The show interface trunk command is useful. You can see if an interface is in trunk mode, which trunk encapsulation protocol it uses (802.1Q or ISL), and what the native VLAN is. We can also see that VLANs 1 – 4094 are allowed on this trunk.

We can also see that only VLAN 1 (native VLAN) and VLAN 50 are currently active. Last but not least, you can see which VLANs are in the forwarding state for spanning-tree.

I want to show you one more thing about access and trunk interfaces:

```
SW2#show interface fa0/2 switchport
Name: Fa0/2
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
```

An interface can be in access mode or in trunk mode. The interface above is connected to H2, and you can see that the operational mode is “static access,” which means it’s in access mode.

```
SW2#show interfaces fa0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
```

This is our trunk interface which is connected to SW1. You can see the operational mode is trunk mode.

```
SW2(config-if)#switchport mode ?
  access        Set trunking mode to ACCESS unconditionally
  dot1q-tunnel  set trunking mode to TUNNEL unconditionally
  dynamic       Set trunking mode to dynamically negotiate access or trunk 
  private-vlan  Set private-vlan mode
  trunk         Set trunking mode to TRUNK unconditionally
```

If I go to the interface configuration to change the `switchport` mode, you can see I have more options than access or trunk mode. There is also a dynamic method. Don’t worry about the other options for now.

```
SW2(config-if)#switchport mode dynamic ?
  auto       Set trunking mode dynamic negotiation parameter to AUTO
  desirable  Set trunking mode dynamic negotiation parameter to DESIRABLE
```

We can choose between dynamic auto and dynamic desirable. Our switch will automatically determine if the interface should become access or trunk port. So what’s the difference between dynamic auto and dynamic desirable? Let’s find out!

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

I’m going to play with the `switchport` mode on SW1 and SW2, and we’ll see the result.

```
SW1(config)#interface fa0/14
SW1(config-if)#switchport mode dynamic auto
```

```
SW2(config)#interface fa0/14
SW2(config-if)#switchport mode dynamic auto
```

First, I’ll change both interfaces to dynamic auto.

```
SW1(config-if)#do show interface f0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
```

```
SW2(config-if)#do show interface f0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
```

Our administrative mode is dynamic auto, and as a result, we now have an access port.

```
SW1(config)#interface fa0/14
SW1(config-if)#switchport mode dynamic desirable
```

```
SW2(config)#interface fa0/14
SW2(config-if)#switchport mode dynamic desirable
```

```
SW1#show interfaces fa0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic desirable
Operational Mode: trunk
```

```
SW2#show interfaces fa0/14 switchport 
Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic desirable
Operational Mode: trunk
```

Once we change both interfaces to dynamic desirable, we end up with a trunk link. What do you think will happen if we mix the switchport types? Maybe dynamic auto on one side and dynamic desirable on the other side? Let’s find out!

```
SW1(config)#interface fa0/14
SW1(config-if)#switchport mode dynamic desirable
```

```
SW2(config)#interface fa0/14
SW2(config-if)#switchport mode dynamic auto
```

```
SW1#show interfaces f0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic desirable
Operational Mode: trunk
```

```
SW2#show interfaces fa0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: trunk
```

It seems our switch has a strong desire to become a trunk. Let’s see what happens with other combinations!

```
SW1(config)#interface fa0/14
SW1(config-if)#switchport mode dynamic auto
```

```
SW2(config)#interface fa0/14
SW2(config-if)#switchport mode trunk
```

```
SW1#show interfaces f0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: trunk
```

```
SW2#show interfaces fa0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
```

Dynamic auto will prefer to become an access port, but if the other interface has been configured as a trunk we will end up with a trunk.

```
SW1(config)#interface fa0/14
SW1(config-if)#switchport mode dynamic auto
```

```
SW2(config)#interface fa0/14
SW2(config-if)#switchport mode access
```

```
SW1#show interfaces f0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
```

```
SW2#show interfaces fa0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
```

Configuring one side as dynamic auto and the other as access will result in an access port.

```
SW1(config)#interface fa0/14
SW1(config-if)#switchport mode dynamic desirable
```

```
SW2(config)#interface fa0/14
SW2(config-if)#switchport mode trunk
```

```
SW1#show interfaces f0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: dynamic desirable
Operational Mode: trunk
```

```
SW2#show interfaces fa0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
```

Dynamic desirable and trunk mode offers us a working trunk.

What do you think will happen if I set one interface in access mode and the other one as trunk? Doesn’t sound like a good idea but let’s push our luck:

```
SW1(config)#interface fa0/14
SW1(config-if)#switchport mode access
```

```
SW2(config)#interface fa0/14
SW2(config-if)#switchport mode trunk
```

```
SW1#show interfaces f0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: static access
Operational Mode: trunk
```

```
SW2#show interfaces fa0/14 switchport
Name: Fa0/14
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
```

```
SW1#
%SPANTREE-7-RECV_1Q_NON_TRUNK: Received 802.1Q BPDU on non trunk FastEthernet0/14 VLAN1.
%SPANTREE-7-BLOCK_PORT_TYPE: Blocking FastEthernet0/14 on VLAN0001. Inconsistent port type.
%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking FastEthernet0/14 on VLAN0001. Port consistency restored.
```

As soon as I change the `switchport` mode, I see these spanning-tree error messages on SW1. Spanning-tree is a protocol that runs on switches that prevent loops in our network.

Let me give you an overview of the different `switchport` modes and the result:

<table><thead><tr><th width="158"></th><th width="80">Trunk</th><th width="81">Access</th><th width="132">Dynamic Auto</th><th width="161">Dynamic Desirable</th></tr></thead><tbody><tr><td>Trunk</td><td>Trunk</td><td>Limited</td><td>Trunk</td><td>Trunk</td></tr><tr><td>Access</td><td>Limited</td><td>Access</td><td>Access</td><td>Access</td></tr><tr><td>Dynamic Auto</td><td>Trunk</td><td>Access</td><td>Access</td><td>Trunk</td></tr><tr><td>Dynamic Desirable</td><td>Trunk</td><td>Access</td><td>Trunk</td><td>Trunk</td></tr></tbody></table>

That’s all I have for you now about trunking. I hope this was useful to you. It’s best if you try some of these commands on your switches so that you become familiar with the different commands. If you enjoyed this lesson, please leave a comment or share it with your friends!

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="SW1" %}
```
hostname SW1
!
vlan 50
 name Computers
!
interface FastEthernet0/1
 switchport access vlan 50
!
interface FastEthernet0/14
 switchport mode trunk
 switchport trunk encapsulation dot1q
!
end
```
{% endtab %}

{% tab title="SW2" %}
```
hostname SW2
!
vlan 50
 name Computers
!
interface FastEthernet0/2
 switchport access vlan 50
!
interface FastEthernet0/14
 switchport mode trunk
 switchport trunk encapsulation dot1q
!
end
```
{% endtab %}
{% endtabs %}
