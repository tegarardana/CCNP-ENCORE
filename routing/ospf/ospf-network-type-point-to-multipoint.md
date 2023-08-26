# OSPF Network Type Point-to-Multipoint

In previous lessons, I explained the OSPF [non-broadcast](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-non-broadcast-network-type-over-frame-relay) and [broadcast ](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-broadcast-network-type-over-frame-relay)network types. Now we are going to look at the OSPF **point-to-multipoint network type**. This is the topology that we will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-network-type-topology.png" alt=""><figcaption></figcaption></figure>

There are a couple of things that you need to be aware of:

* Automatic neighbor discovery, so there is no need to configure OSPF neighbors yourself.
* No DR/BDR election since OSPF sees the network as a collection of point-to-point links.
* Only a single IP subnet is used in the topology above.
* Make sure your frame-relay network is configured with the broadcast keyword.

Let’s take a look at the configuration:

<pre><code><strong>Hub(config)#interface serial 0/0
</strong><strong>Hub(config-if)#ip address 192.168.123.1 255.255.255.0
</strong><strong>Hub(config-if)#encapsulation frame-relay
</strong><strong>Hub(config-if)#ip ospf network point-to-multipoint
</strong><strong>Hub(config-if)#exit
</strong><strong>Hub(config)#router ospf 1
</strong><strong>Hub(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

This is the hub configuration. I changed the network type to point-to-multipoint and advertised the 192.168.123.0 /24 network in OSPF. Let’s look at the spoke router configuration:

<pre><code><strong>Spoke1(config)#interface serial 0/0
</strong><strong>Spoke1(config-if)#ip address 192.168.123.2 255.255.255.0
</strong><strong>Spoke1(config-if)#encapsulation frame-relay 
</strong><strong>Spoke1(config-if)#ip ospf network point-to-multipoint
</strong><strong>Spoke1(config-if)#exit
</strong><strong>Spoke1(config)#router ospf 1
</strong><strong>Spoke1(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>Spoke2(config)#interface serial 0/0
</strong><strong>Spoke2(config-if)#ip address 192.168.123.3 255.255.255.0
</strong><strong>Spoke2(config-if)#encapsulation frame-relay 
</strong><strong>Spoke2(config-if)#ip ospf network point-to-multipoint
</strong><strong>Spoke2(config-if)#exit
</strong><strong>Spoke2(config)#router ospf 1
</strong><strong>Spoke2(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

Just a basic configuration. You’ll have to change the OSPF network type and type in the correct network statements to become OSPF neighbors.

{% hint style="info" %}
Don’t forget that you’ll require the **broadcast** keyword for your frame-relay maps, or this will not work. By default Inverse ARP will do this, but if you disabled Inverse ARP, you’ll have to create the correct frame-relay maps yourself.
{% endhint %}

Let’s check if we have OSPF neighbors:

<pre><code><strong>Hub#show ip ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.123.3     0   FULL/  -        00:01:35    192.168.123.3   Serial0/0
<strong>192.168.123.2     0   FULL/  -        00:01:56    192.168.123.2   Serial0/0
</strong></code></pre>

You can see that the hub router has two OSPF neighbors and there is no DR/BDR election.

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
 ip ospf network point-to-multipoint
 clock rate 2000000
!
router ospf 1
 network 192.168.123.0 0.0.0.255 area 0
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
 ip ospf network point-to-multipoint
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
 ip ospf network point-to-multipoint
 clock rate 2000000
!
router ospf 1
 network 192.168.123.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

That’s it for now. I hope you enjoyed this example! Feel free to ask any questions.
