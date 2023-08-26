# EIGRP Passive Interface

When you use the EIGRP network command, two things will happen:

* All interfaces that have a network that falls within the range of your network command will be advertised in EIGRP.
* EIGRP hello packets will be sent on these interfaces.

Sometimes however you might want to advertise a network in EIGRP but you don’t want to send hello packets everywhere. Take a look at the topology below for an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/eigrp-passive-interface-demo-topology.png" alt=""><figcaption></figcaption></figure>

Above we have two routers, R1 and R2. On the left side there’s the 192.168.10.0 /24 network with a switch and some computers. R1 wants to advertise this network to R2 but since there are no other EIGRP routers in the 192.168.10.0 /24 network, it’s pointless to send EIGRP hello packets on the FastEthernet 0/1 interface.

It’s also a security risk, when someone connects a router in the 192.168.10.0 /24 network (or starts a virtual router on their computer) they will be able to become EIGRP neighbors with R1.

To prevent this from happening, we will use the **passive-interface** command. This will ensure that the network is advertised in EIGRP but it will disable hello packets on the interface.

{% hint style="info" %}
Another trick to advertise something in EIGRP without sending hello packets on the interface is using the “redistribute” command.
{% endhint %}

Let me show you how to configure this.

## Configuration

Here’s the EIGRP configuration of R1 and R2:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#network 192.168.12.0
</strong><strong>R1(config-router)#network 192.168.10.0 
</strong></code></pre>

<pre><code><strong>R2(config)#router eigrp 12
</strong><strong>R2(config-router)#network 192.168.12.0
</strong></code></pre>

As a result, R2 will learn network 192.168.10.0 /24:

<pre><code><strong>R2#show ip route eigrp 
</strong>D    192.168.10.0/24 [90/307200] via 192.168.12.1, 00:00:59, FastEthernet0/0
</code></pre>

The problem however is that R1 is sending hello packets towards our computers. You can verify this by enabling a debug:

<pre><code><strong>R1#debug eigrp packets hello
</strong>EIGRP Packets debugging is on
    (HELLO)

EIGRP: Sending HELLO on FastEthernet0/0
  AS 12, Flags 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0

EIGRP: Sending HELLO on FastEthernet0/1
  AS 12, Flags 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0
</code></pre>

Above you can see that the hello packets are going in both directions.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/eigrp-sending-hello-packets.png" alt=""><figcaption></figcaption></figure>

Let’s use the passive interface command to disable the hello packets towards the switch:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#passive-interface FastEthernet 0/1
</strong></code></pre>

That’s all you have to do. You can find all passive interfaces with the following command:

<pre><code><strong>R1#show ip protocols 
</strong>Routing Protocol is "eigrp 12"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP metric weight K1=1, K2=0, K3=1, K4=0, K5=0
  EIGRP maximum hopcount 100
  EIGRP maximum metric variance 1
  Redistributing: eigrp 12
  EIGRP NSF-aware route hold timer is 240s
  Automatic network summarization is in effect
  Automatic address summarization:
    192.168.10.0/24 for FastEthernet0/0
  Maximum path: 4
  Routing for Networks:
    192.168.10.0
    192.168.12.0
  Passive Interface(s):
    FastEthernet0/1
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: internal 90 external 170
</code></pre>

If you left the debug enabled you will see that hello packets are now only sent towards R2:

<pre><code><strong>EIGRP: Sending HELLO on FastEthernet0/0
</strong>  AS 12, Flags 0x0, Seq 0/0 idbQ 0/0 iidbQ un/rely 0/0
</code></pre>

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/eigrp-passive-interface-supress-hello.png" alt=""><figcaption></figcaption></figure>

The network is still advertised which we can confirm by checking R2:

<pre><code><strong>R2#show ip route eigrp 
</strong>D    192.168.10.0/24 [90/307200] via 192.168.12.1, 00:02:54, FastEthernet0/0
</code></pre>

Problem solved. The network is still advertised but we don’t send any EIGRP hello packets anymore towards our computers. You should use this command on all interfaces where no EIGRP neighbors are found and that have a network which is advertised in EIGRP.

If you have many interfaces that should be passive then you can also use the **passive-interface default** command. This will disable the sending of hello packets on all interfaces, if you do want to send hello packets then you need to use the **no passive-interface** command for these interfaces.

{% hint style="info" %}
RIP and OSPF also support the passive interface command, it works similar to EIGRP. The only difference with RIP is that it doesn’t use hello packets, it just suppresses the advertisement of RIP packets on the interface.
{% endhint %}

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet0/0
 ip address 192.168.10.254 255.255.255.0
!
interface FastEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
router eigrp 12
 network 192.168.12.0
 network 192.168.10.0
 passive-interface FastEthernet0/1
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
router eigrp 12
 network 192.168.12.0
!
end
```
{% endtab %}
{% endtabs %}

That’s all there is to it. If you have any questions, feel free to leave a comment!
