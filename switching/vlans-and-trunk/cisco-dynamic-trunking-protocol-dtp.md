# Cisco Dynamic Trunking Protocol (DTP)

In this lesson, we’ll take a look at DTP (Dynamic Trunking Protocol) negotiation. DTP is normally used on Cisco IOS switches to negotiate if the interface should become an access port or trunk.

By default, DTP is enabled, and the interfaces of your switches will be in “dynamic auto” or “dynamic desirable” mode. This means that your interface will be in trunk mode whenever you receive a DTP packet that requests to form a trunk. If you are unfamiliar with DTP and the different interface settings, then you might want to read my [“How to configure Trunk on Cisco Catalyst Switch”](https://networklessons.com/switching/how-to-configure-trunk-on-cisco-catalyst-switch/) lesson before continuing.

Let’s take a look at DTP negotiation and how to disable it. I’ll be using two switches for this:

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

I didn’t configure anything on my switches. Let’s see what the default settings are:

<pre><code><strong>SW1#show interfaces fa0/24 switchport        
</strong>Name: Fa0/24
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: On
</code></pre>

<pre><code><strong>SW2#show interfaces fastEthernet 0/24 switchport 
</strong>Name: Fa0/24
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: On
</code></pre>

Without configuring anything on the interfaces, we are using **`dynamic auto`** mode, and as a result, the interfaces are in **`access`** mode.

{% hint style="info" %}
Depending on the switch model and IOS version, the default might be `dynamic auto` or `dynamic desirable`. The switches in my example are Cisco Catalyst 3560 switches.
{% endhint %}

There are two ways to disable DTP negotiation:

* Configure the interface for access mode.
* Use the **`switchport nonegotiate`** command on the interface.

Configuring the interface for trunking does not disable DTP negotiation. Let me give you an example. First, we’ll configure the interfaces for access mode:

<pre><code><strong>SW1(config)#interface fastEthernet 0/24
</strong><strong>SW1(config-if)#switchport mode access
</strong></code></pre>

<pre><code><strong>SW2(config)#interface fastEthernet 0/24
</strong><strong>SW2(config-if)#switchport mode access 
</strong></code></pre>

When we look again at the `switchport` settings, we can see that DTP negotiation is now disabled:

<pre><code><strong>SW1#show interfaces fastEthernet 0/24 switchport 
</strong>Name: Fa0/24
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
</code></pre>

So configuring an interface yourself to use access mode disables DTP negotiation. How about creating a trunk ourselves?

<pre><code><strong>SW1(config)#interface fastEthernet 0/24
</strong><strong>SW1(config-if)#switchport trunk encapsulation dot1q 
</strong><strong>SW1(config-if)#switchport mode trunk
</strong></code></pre>

<pre><code><strong>SW2(config)#interface fastEthernet 0/24
</strong><strong>SW2(config-if)#switchport trunk encapsulation dot1q 
</strong><strong>SW2(config-if)#switchport mode trunk 
</strong></code></pre>

Does this mean that DTP negotiation will also be disabled?

<pre><code><strong>SW1#show interfaces fastEthernet 0/24 switchport | include Negotiation
</strong>Negotiation of Trunking: On
</code></pre>

Unfortunately not. If you configure a trunk yourself, DTP negotiation is still enabled. We can disable it, but there’s another command we have to use:

<pre><code><strong>SW1(config)#interface fastEthernet 0/24
</strong><strong>SW1(config-if)#switchport nonegotiate 
</strong></code></pre>

<pre><code><strong>SW2(config)#interface fastEthernet 0/24          
</strong><strong>SW2(config-if)#switchport nonegotiate 
</strong></code></pre>

This disables DTP for trunk interfaces. Let’s verify it:

<pre><code><strong>SW1#show interfaces fastEthernet 0/24 switchport | include Negotiation
</strong>Negotiation of Trunking: Off
</code></pre>

Now it’s disabled! You have now learned the two methods to disable DTP negotiation. If you have any questions, feel free to leave a comment.
