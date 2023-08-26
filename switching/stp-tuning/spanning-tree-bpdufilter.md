# Spanning-Tree BPDUFilter

The spanning-tree BPDUfilter works similar to [BPDUGuard](https://networklessons.com/cisco/ccnp-encor-350-401/spanning-tree-bpduguard) as it allows you to block malicious BPDUs. The difference is that BPDUguard will put the interface that it receives the BPDU on in err-disable mode while BPDUfilter just “filters” it. In this lesson we’ll take a good look at how BPDUfilter works.

BPDUfilter can be configured **globally** or on the **interface** **level** and there’s a difference:

* Global: if you enable BPDUfilter globally then any interface with portfast enabled will not send or receive any BPDUs. When you receive a BPDU on a portfast enabled interface then it will lose its portfast status, disables BPDU filtering and acts as a normal interface.
* Interface: if you enable BPDUfilter on the interface it will **ignore** incoming BPDUs and it will **not send** any BPDUs. This is the equivalent of disabling spanning-tree.

You have to be careful when you enable BPDUfilter on interfaces. You can use it on interfaces in access mode that connect to computers but make sure you never configure it on interfaces connected to other switches; if you do you might end up with a loop.

Let’s use the following topology to demonstrate the BPDUfilter:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/spanning-tree-bpduguard-topology.png" alt=""><figcaption></figcaption></figure>

I’m going to use SW2 and SW3 to demonstrate BPDUfilter:

<pre><code><strong>SW2(config)#interface fa0/16
</strong><strong>SW2(config-if)#spanning-tree portfast trunk
</strong><strong>SW2(config-if)#spanning-tree bpdufilter enable
</strong></code></pre>

It will stop sending BPDUs and it will ignore whatever is received. Let’s enable a debug to see what it does:

<pre><code><strong>SW2#debug spanning-tree bpdu
</strong></code></pre>

You won’t see any exciting messages but if you enable BPDU debugging you’ll notice that it doesn’t send any BPDUs anymore. If you want you can also enable BPDU debugging on SW3 and you’ll see that you won’t receive any from SW2.

<pre><code><strong>SW2(config)#interface fa0/16
</strong><strong>SW2(config-if)#no spanning-tree bpdufilter enable
</strong></code></pre>

Let’s get rid of the BPDUfilter command on the interface level and enable it globally:

<pre><code><strong>SW2(config)#spanning-tree portfast bpdufilter default
</strong></code></pre>

You can also use the global command for BPDUfilter. This will enable BPDUfilter on all interfaces that have portfast.

That’s all there is to it. Personally I wouldn’t use this and use BPDUguard instead. If you don’t expect BPDUs on an interface then it’s better to get a notification (through err-disable) then not seeing what is going on…
