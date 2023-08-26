# Spanning-Tree UDLD and LoopGuard

If you ever used fiber cables you might have noticed that there is a different connector to transmit and receive traffic.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/fiber-cable-connectors-300x199.jpg" alt=""><figcaption></figcaption></figure>

If one of the cables (transmit or receive) fails we’ll have a **unidirectional link failure** and this can cause spanning tree loops. There are two protocols that can take care of this problem:

* LoopGuard
* UDLD ( Unidirectional Link Detection )

Let’s start by taking a close look at what will happen if we have a unidirectional link failure:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/switches-fiber-connections-topology.png" alt=""><figcaption></figcaption></figure>

Imagine the links between the switches are fiber links. In reality there’s a different connector for transmit and receive. SW3 is receiving BPDUs from SW2 and as a result the interface has become an alternate port and is in blocking mode.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/spanning-tree-unidirectional-link-failure.png" alt=""><figcaption></figcaption></figure>

Now something goes wrong…the transmit connector on SW2 towards SW3 was eaten by mice failed due to unknown reasons. As a result SW3 is not receiving any BPDUs from SW2 but it can still send traffic to SW2.

Because SW3 is not receiving anymore BPDUs on its alternate port it will go into forwarding mode. We now have a **one way loop** as indicated by the green arrow.

One of the methods we can use to solve our unidirectional link failure is to configure **LoopGuard.** When a switch is sending but not receiving BPDUs on the interface, LoopGuard will place the interface in the **loop-inconsistent state** and block all traffic:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/spanning-tree-loop-guard-example.png" alt=""><figcaption></figcaption></figure>

Let’s take a look what this looks like on actual switches. I will use the same topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-example-with-ports.png" alt=""><figcaption></figcaption></figure>

Let’s enable loopguard:

<pre><code><strong>SW1(config)#spanning-tree loopguard default
</strong></code></pre>

<pre><code><strong>SW2(config)#spanning-tree loopguard default
</strong></code></pre>

<pre><code><strong>SW3(config)#spanning-tree loopguard default
</strong></code></pre>

Use the spanning-tree loopguard default command to enable LoopGuard globally. I don’t have any fiber connectors so I’m unable to create a unidirectional link failure. I can simulate it however by using BPDUfilter on SW2’s fa0/16 interface. SW3 won’t receive any BPDUs anymore on its alternate port which will cause it to go into forwarding mode:

<pre><code><strong>SW2(config)#interface fa0/16
</strong><strong>SW2(config-if)#spanning-tree portfast trunk
</strong><strong>SW2(config-if)#spanning-tree bpdufilter enable
</strong></code></pre>

Here’s what will happen:

```
SW3# 
*Mar  1 00:17:14.431: %SPANTREE-2-LOOPGUARD_BLOCK: Loop guard blocking port FastEthernet0/16 on VLAN0001.
```

Normally this would cause a loop but luckily we have LoopGuard configured. You can see this error message appearing in your console, problem solved!

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="SW1" %}
```
hostname SW1
!
spanning-tree loopguard default
!
end
```
{% endtab %}

{% tab title="SW2" %}
```
hostname SW2
!
spanning-tree loopguard default
!
interface FastEthernet0/16
 spanning-tree portfast trunk
 spanning-tree bpdufilter enable
!
end
```
{% endtab %}

{% tab title="SW3" %}
```
hostname SW3
!
spanning-tree loopguard default
!
end
```
{% endtab %}
{% endtabs %}

Want to take a look for yourself? Here you will find the final configuration of each device.\
If you want you don’t have to configure LoopGuard globally, you can also do it on the interface level like this:

<pre><code><strong>SW3(config-if)#spanning-tree guard loop
</strong></code></pre>

The other protocol we can use to deal with unidirectional link failures is called **UDLD (UniDirectional Link Detection).** This protocol is not part of the spanning tree toolkit but it does help us to prevent loops.

Simply said UDLD is a layer 2 protocol that works like a keepalive mechanism. You send hello messages, you receive them and life is good. As soon as you still send hello messages but don’t receive them anymore you know something is wrong and we’ll block the interface.

Let’s use the same topology but configure UDLD this time. Don’t forget to get rid of loopguard first…

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/rapid-spanning-tree-example-with-ports.png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>SW1(config)#udld ?      
</strong>  aggressive  Enable UDLD protocol in aggressive mode on fiber ports except
              where locally configured
  enable      Enable UDLD protocol on fiber ports except where locally
              configured
  message     Set UDLD message parameters
</code></pre>

There are a number of methods how you can configure UDLD. You can do it globally with the udld command but this will only activate UDLD **for fiber links**!

There are two options for UDLD:

* **Normal** (default)
* **Aggressive**

When you set UDLD to **normal** it will mark the port as **undetermined** but it won’t shut the interface when something is wrong. This is only used to “inform” you but it won’t stop loops.

**Aggressive** is a better solution. When it loses connectivity to a neighbor it will send a UDLD frame out once a second for 8 seconds. If the neighbor does not respond the interface will be put in **err-disable** mode.

Let’s use two switches to demonstrate UDLD:

Let’s enable UDLD:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/spanning-tree-bpduguard-topology.png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>SW2(config)#interface fa0/16
</strong><strong>SW2(config-if)#udld port aggressive
</strong></code></pre>

<pre><code><strong>SW3(config)#interface fa0/16
</strong><strong>SW3(config-if)#udld port aggressive
</strong></code></pre>

We’ll use SW2 and SW3 to demonstrate UDLD. I’ll use aggressive mode so we can see that the interface goes down when something is wrong. To see what is going on in real time we’ll use a debug:

<pre><code><strong>SW2#debug udld events 
</strong>UDLD events debugging is on
</code></pre>

```
SW3#
New_entry = 34422DC (Fa0/16)
Found an entry from same device (Fa0/16)
Cached entries = 2 (Fa0/16)
Entry (0x242BB9C) deleted: 1 entries cached
Cached entries = 1 (Fa0/16)
Checking if multiple neighbors (Fa0/16)
Single neighbor detected (Fa0/16)
Checking if link is bidirectional (Fa0/16)
Found my own ID pair in 2way conn list (Fa0/16)
```

Now the tricky part will be to simulate a unidirectional link failure. LoopGuard was easier because it was based on BPDUs. UDLD runs its own layer 2 protocol by using the proprietary MAC address 0100.0ccc.cccc. We can create a filter to block the UDLD traffic:

<pre><code><strong>SW3(config)#mac access-list extended UDLD-FILTER
</strong><strong>SW3(config-ext-macl)#deny any host 0100.0ccc.cccc
</strong><strong>SW3(config-ext-macl)#permit any any
</strong><strong>SW3(config-ext-macl)#exit
</strong><strong>SW3(config)#interface fa0/16
</strong><strong>SW3(config-if)#mac access-group UDLD-FILTER in
</strong></code></pre>

This is a creative way to cause trouble. By filtering the MAC address of UDLD on one side it will think that there is an unidirectional link failure! Here’s what you will see:

```
SW2#
UDLD FSM updated port, bi-flag udld_empty_echo, phase udld_detection (Fa0/16)
timeout timer = 0 (Fa0/16)
Phase set to EXT.  (Fa0/16)
New_entry = 370CED0 (Fa0/16)
Found an entry from same device (Fa0/16)
Cached entries = 2 (Fa0/16)
Entry (0x3792BE0) deleted: 1 entries cached
Cached entries = 1 (Fa0/16)
Zero IDs in 2way conn list (Fa0/16)
Zero IDs in 2way conn list (Fa0/16)
UDLD disabled port, packet received in extended detection (Fa0/16)
%UDLD-4-UDLD_PORT_DISABLED: UDLD disabled interface Fa0/16, unidirectional link detected
%PM-4-ERR_DISABLE: udld error detected on Fa0/16, putting Fa0/16 in err-disable state
```

You’ll see a lot of debug information flying by but the end result will be that the port is now in err-disable state. Here’s a show command you can use to check it:

<pre><code><strong>SW2#show udld fastEthernet 0/16
</strong>
Interface Fa0/16
---
Port enable administrative configuration setting: Enabled / in aggressive mode
Port enable operational state: Enabled / in aggressive mode
Current bidirectional state: Unidirectional
Current operational state: Disabled port
</code></pre>

You can verify it by using the **show udld command**.

LoopGuard and UDLD both solve the same problem: Unidirectional Link failures. They have some overlap but there are a number of differences, here’s an overview:

|                                                                             | **LoopGuard**                                           | **UDLD**                                  |
| --------------------------------------------------------------------------- | ------------------------------------------------------- | ----------------------------------------- |
| **Configuration**                                                           | Global / per port                                       | Global (for fiber) / per port             |
| **Per VLAN?**                                                               | Yes                                                     | No, per port                              |
| **Autorecovery**                                                            | Yes                                                     | Yes – requires err-disable timeout.       |
| **Protection against STP failures because of unidirectional link failures** | Yes – need to enable it on all root and alternate ports | Yes – need to enable it on all interfaces |
| **Protection against STP failures because no BPDUs are sent**               | Yes                                                     | No                                        |
| **Protection against miswiring**                                            | No                                                      | Yes                                       |

That’s all there is to it. I hope you enjoyed this lesson about LoopGuard and UDLD.
