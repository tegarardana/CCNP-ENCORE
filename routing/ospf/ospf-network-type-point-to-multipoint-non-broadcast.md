# OSPF Network Type Point-to-Multipoint Non-Broadcast

In a previous lesson, I showed you how to configure the [OSPF point-to-multipoint network type](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-point-to-multipoint-network-type-over-frame-relay). This time we’ll look at the OSPF point-to-multipoint non-broadcast network type. It’s the same thing, but we’ll have to specify OSPF neighbors ourselves. Here is the topology that we’ll use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-network-type-topology.png" alt=""><figcaption></figcaption></figure>

There are a couple of things that you need to be aware of:

* No Automatic neighbor discovery, so you need to configure OSPF neighbors yourself!
* No DR/BDR election since OSPF sees the network as a collection of point-to-point links.
* Only a single IP subnet is used in the topology above.

Here’s the configuration for the Hub router:

<pre><code><strong>Hub(config)#interface serial 0/0
</strong><strong>Hub(config-if)#ip address 192.168.123.1 255.255.255.0
</strong><strong>Hub(config-if)#encapsulation frame-relay
</strong><strong>Hub(config-if)#ip ospf network point-to-multipoint non-broadcast
</strong><strong>Hub(config-if)#exit
</strong><strong>Hub(config)#router ospf 1
</strong><strong>Hub(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong><strong>Hub(config-router)#neighbor 192.168.123.2
</strong><strong>Hub(config-router)#neighbor 192.168.123.3
</strong></code></pre>

This is the hub configuration. I changed the network type to point-to-multipoint non-broadcast, advertised the 192.168.123.0 /24 network in OSPF, and, most important…specified the **OSPF neighbors myself**. Let’s look at the spoke router configuration:

<pre><code><strong>Spoke1(config)#interface serial 0/0
</strong><strong>Spoke1(config-if)#ip address 192.168.123.2 255.255.255.0
</strong><strong>Spoke1(config-if)#encapsulation frame-relay 
</strong><strong>Spoke1(config-if)#ip ospf network point-to-multipoint non-broadcast
</strong><strong>Spoke1(config-if)#exit
</strong><strong>Spoke1(config)#router ospf 1
</strong><strong>Spoke1(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>Spoke2(config)#interface serial 0/0
</strong><strong>Spoke2(config-if)#ip address 192.168.123.3 255.255.255.0
</strong><strong>Spoke2(config-if)#encapsulation frame-relay 
</strong><strong>Spoke2(config-if)#ip ospf network point-to-multipoint non-broadcast
</strong><strong>Spoke2(config-if)#exit
</strong><strong>Spoke2(config)#router ospf 1
</strong><strong>Spoke2(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

Just a basic configuration. You’ll have to change the OSPF network type and type in the correct network statements to become OSPF neighbors. Let’s verify the configuration:

<pre><code><strong>Hub#show ip ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.123.3     0   FULL/  -        00:01:35    192.168.123.3   Serial0/0
192.168.123.2     0   FULL/  -        00:01:56    192.168.123.2   Serial0/0
</code></pre>

As you can see above, the Hub router has two neighbors. There is no DR/BDR election. That’s all there is to it!

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="Hub" %}
```
hostname Hub
!
interface Serial0/0
 ip address 192.168.123.1 255.255.255.0
 encapsulation frame-relay
 ip ospf network point-to-multipoint non-broadcast
 clock rate 2000000
!
router ospf 1
 network 192.168.123.0 0.0.0.255 area 0
 neighbor 192.168.123.3
 neighbor 192.168.123.2
!
end
```
{% endtab %}

{% tab title="Spoke1" %}
```
hostname Spoke1
!
interface Serial0/0
 ip address 192.168.123.2 255.255.255.0
 encapsulation frame-relay
 ip ospf network point-to-multipoint non-broadcast
 clock rate 2000000
!
router ospf 1
 network 192.168.123.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="Spoke2" %}
```
hostname Spoke2
!
interface Serial0/0
 ip address 192.168.123.3 255.255.255.0
 encapsulation frame-relay
 ip ospf network point-to-multipoint non-broadcast
 clock rate 2000000
!
router ospf 1
 network 192.168.123.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

If you have any questions just leave a comment.
