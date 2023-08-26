# OSPF Passive Interface

When you use the network command in OSPF, two things will happen:

* All interfaces that have a network that falls within the range of the network command will be advertised in OSPF.
* OSPF hello packets are sent on these interfaces.

Sometimes it’s undesirable to send OSPF hello packets on certain interfaces. Take a look at the image below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/ospf-passive-interface-lab-topology.png" alt=""><figcaption></figcaption></figure>

R1 and R2 are configured for OSPF. R1 is connected to network 192.168.10 /24 which has some computers connected to a switch. R1 wants to advertise this network to R2.

Once we use the network command to advertise 192.168.10.0 /24 in OSPF, R1 will also send OSPF hello packets towards the switch. This is a bad idea, first of all because there are no routers on this network but it’s also a security risk. If someone on the computer starts an application that replies with OSPF hello packets then R1 will try to become neighbors. An attacker could advertise fake routes using this technique.

To prevent this from happening, we can use the **passive-interface** command. This command tells OSPF not to send hello packets on certain interfaces. Let’s see how it works…

## Configuration

Here’s the OSPF configuration of R1 and R2:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong><strong>R1(config-router)#network 192.168.10.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>R2(config)#router ospf 1
</strong><strong>R2(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong></code></pre>

With the above configuration, R2 will learn network 192.168.10.0 /24:

<pre><code><strong>R2#show ip route ospf 
</strong>O    192.168.10.0/24 [110/20] via 192.168.12.1, 00:03:21, FastEthernet0/0
</code></pre>

This is great but a side-effect of this configuration is that R1 will send hello packets on its FastEthernet 0/1 interface. We can see this with a debug:

<pre><code><strong>R1#debug ip ospf hello 
</strong>OSPF hello events debugging is on

OSPF: Send hello to 224.0.0.5 area 0 on FastEthernet0/1 from 192.168.10.254

OSPF: Send hello to 224.0.0.5 area 0 on FastEthernet0/0 from 192.168.12.1
</code></pre>

Above you can see that hello packets are sent in both directions.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/eigrp-sending-hello-packets.png" alt=""><figcaption></figcaption></figure>

Let’s fix this. We will configure OSPF to stop the hello packets towards the switch:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#passive-interface FastEthernet 0/1
</strong></code></pre>

You only have to use the **passive-interface** command under the OSPF process. You can verify our work with the following command:

<pre><code><strong>R1#show ip protocols 
</strong>Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 192.168.12.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.10.0 0.0.0.255 area 0
    192.168.12.0 0.0.0.255 area 0
 Reference bandwidth unit is 100 mbps
  Passive Interface(s):
    FastEthernet0/1
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 110)
</code></pre>

Show ip protocols will tell us which interfaces are configured as passive interface(s). If you left the debug enabled you will see that the hello packets are blocked:

```
R1#
OSPF: Send hello to 224.0.0.5 area 0 on FastEthernet0/0 from 192.168.12.1
```

That’s looking good, they are only sent towards R2 now.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/eigrp-passive-interface-supress-hello.png" alt=""><figcaption></figcaption></figure>

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet 0/1
 ip address 192.168.10.254 255.255.255.0
!
interface FastEthernet 0/0
 ip address 192.168.12.1 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.10.0 0.0.0.255 area 0
 passive-interface FastEthernet 0/1
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface FastEthernet 0/0
 ip address 192.168.12.2 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255
!
end
```
{% endtab %}
{% endtabs %}

If you have many interfaces then it might be annoying to configure each of them as a passive interface. For example let’s imagine that R1 is used as a [router on a stick](https://networklessons.com/ospf/how-to-configure-router-on-a-stick) for VLANs that are configured on the switch. It will have many sub-interfaces, on each of those it will send OSPF hello packets:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/ospf-sub-interfaces.png" alt=""><figcaption></figcaption></figure>

We could use the passive-interface command for each of these sub-interfaces but there’s a better solution for this:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#passive-interface default
</strong><strong>R1(config-router)#no passive-interface FastEthernet 0/0
</strong></code></pre>

The configuration above will make all interfaces passive and you have to tell the router which interfaces should send OSPF hello packets. This is easier and it will prevent OSPF from sending hello packets when someone creates a new sub-interface and forgets to make it passive.\


{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet 0/1
 ip address 192.168.10.254 255.255.255.0
!
interface FastEthernet 0/0
 ip address 192.168.12.1 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.10.0 0.0.0.255 area 0
 passive-interface default
 no passive-interface FastEthernet 0/0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface FastEthernet 0/0
 ip address 192.168.12.2 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
RIP and EIGRP also support the passive-interface command. It works similarly for EIGRP but RIP works a bit differently. It doesn’t create neighbor adjacencies so it just suppresses route advertisements on the passive interface.
{% endhint %}

Hopefully these examples have been useful, if you have any questions feel free to leave a comment!
