# OSPF Network Type Broadcast

In a previous lesson, I explained the [OSPF Non-Broadcast Network Type](https://networklessons.com/cisco/ccnp-encor-350-401/ospf-non-broadcast-network-type-over-frame-relay). Now it’s time for the broadcast network type. If you understand non-broadcast, then this one is easy. It’s the EXACT same thing, except we don’t have to configure neighbors. OSPF will use multicast and discover OSPF neighbors automatically. The broadcast network type is the default for Ethernet interfaces.

This is the topology that we’ll use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-network-type-topology.png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>Hub(config)#interface serial 0/0
</strong><strong>Hub(config-if)#ip address 192.168.123.1 255.255.255.0
</strong><strong>Hub(config-if)#encapsulation frame-relay
</strong><strong>Hub(config-if)#ip ospf network broadcast
</strong><strong>Hub(config-if)#exit
</strong><strong>Hub(config)#router ospf 1
</strong><strong>Hub(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

Here is the configuration of the Hub router. I used the `ip ospf network broadcast` command to change the OSPF network type. Here are the spoke routers:

<pre><code><strong>Spoke1(config)#interface serial 0/0
</strong><strong>Spoke1(config-if)#ip address 192.168.123.2 255.255.255.0
</strong><strong>Spoke1(config-if)#encapsulation frame-relay 
</strong><strong>Spoke1(config-if)#ip ospf network broadcast 
</strong><strong>Spoke1(config-if)#ip ospf priority 0
</strong><strong>Spoke1(config-if)#exit
</strong><strong>Spoke1(config)#router ospf 1
</strong><strong>Spoke1(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>Spoke2(config)#interface serial 0/0
</strong><strong>Spoke2(config-if)#ip address 192.168.123.3 255.255.255.0
</strong><strong>Spoke2(config-if)#encapsulation frame-relay 
</strong><strong>Spoke2(config-if)#ip ospf network broadcast 
</strong><strong>Spoke2(config-if)#ip ospf priority 0
</strong><strong>Spoke2(config-if)#exit
</strong><strong>Spoke2(config)#router ospf 1
</strong><strong>Spoke2(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

Make sure you set the priority to 0 for the spoke routers. We don’t want them to become the DR or BDR!

{% hint style="info" %}
Make sure you have a frame-relay map statement with the broadcast keyword, or you won’t be able to send multicast on your frame-relay network. By default Inverse ARP is enabled and will do this for you…if you don’t have inverse ARP, make sure you add it!
{% endhint %}

<pre><code><strong>Hub#show ip ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Address         Interface
<strong>192.168.123.2     0   FULL/DROTHER    00:00:30    192.168.123.2   Serial0/0
</strong><strong>192.168.123.3     0   FULL/DROTHER    00:00:35    192.168.123.3   Serial0/0
</strong></code></pre>

Here you can see that the hub router is the DR because the spoke routers are DROTHERS.

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
 ip ospf network broadcast
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
 ip ospf network broadcast
 ip ospf priority 0
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
 ip ospf network broadcast
 ip ospf priority 0
 clock rate 2000000
!
router ospf 1
 network 192.168.123.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

That’s all I wanted to show you. If you have any more questions, please let me know!
