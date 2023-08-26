# Spanning-Tree BPDUGuard

Spanning-tree BPDUguard is one of the features that helps you protect your spanning-tree topology. Let me give you an example:

<figure><img src="../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

In my topology above we have a perfectly working spanning tree topology. By default spanning tree will send and receive BPDUs on all interfaces. In our example we have a computer on the fa0/2 interface of SW2. Someone with curious hostile intentions could start a tool that generates BPDUs with a superior bridge ID. What’ll happen is that our switches will believe that the root bridge can now be reached through SW2 and we’ll have a spanning tree re-calculation. Doesn’t sound like a good idea right? Here’s what could go wrong:

<figure><img src="../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

You could even do a man in the middle attack without anyone knowing. Imagine I connect my computer to two switches. If I become the root bridge all traffic from SW1 or SW3 towards SW2 will flow through me. I’ll run Wireshark and wait till the magic happens.

We can use BPDUGuard to prevent this from happening as it will block BPDUs:

<figure><img src="../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

BPDUguard will ensure that when we receive a BPDU on an interface that the interface will go into **`err-disable mode`**.

Let’s take a look how to configure this…

### Configuration

I will use the following topology:

<figure><img src="../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

To demonstrate BPDUguard I’m going to use two switches. I’ll configure the fa0/16 interface of SW2 so it will go into err-disable mode if it receives a BPDU from SW3.

<pre><code><strong>SW2(config)#interface fa0/16
</strong><strong>SW2(config-if)#spanning-tree bpduguard enable
</strong></code></pre>

This is how you enable it on the interface. Keep in mind normally you will never do this between switches; you should configure this on the interfaces in access mode that connect to computers.

```
SW2#
%SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU on port Fa0/16 with BPDU Guard enabled. Disabling port.
%PM-4-ERR_DISABLE: bpduguard error detected on Fa0/16, putting Fa0/16 in err-disable state
: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/16, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to down
*Mar  1 00:19:32.089: %LINK-3-UPDOWN: Interface FastEthernet0/16, changed state to down
```

Uh oh…there goes our interface.

<pre><code><strong>SW2(config-if)#no spanning-tree bpduguard 
</strong><strong>SW2(config-if)#shutdown
</strong><strong>SW2(config-if)#no shutdown
</strong></code></pre>

Get rid of BPDUguard and do a shut/no shut to get the interface back up and running.

<pre><code><strong>SW2(config)#spanning-tree portfast bpduguard default
</strong></code></pre>

You can also use the `spanning-tree portfast bpduguard` default command. This will globally activate BPDUguard on all interfaces that have portfast enabled.

<pre><code><strong>SW2(config)#spanning-tree portfast default
</strong></code></pre>

Portfast can also be enabled globally for all interfaces running in access mode.

<pre><code><strong>SW2#show spanning-tree summary        
</strong>Switch is in pvst mode
Root bridge for: none
Extended system ID           is enabled
Portfast Default             is enabled
PortFast BPDU Guard Default  is enabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is enabled
UplinkFast                   is disabled
BackboneFast                 is disabled
Configured Pathcost method used is short
</code></pre>

Here’s a useful command so you can verify your configuration. You can see that portfast and BPDUGuard have been enabled globally.

That’s all there to it. I hope you enjoyed this lesson!
