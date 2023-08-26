# OSPF Network Type Point-to-Point

In a number of lessons, I covered the OSPF network types. This lesson is the final one and will cover the OSPF Point-to-Point Network Type. I will be using a frame-relay point-to-point topology to demonstrate it, here it is:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/frame-relay-p2p-lab-topology-serial.png" alt=""><figcaption></figcaption></figure>

Here’s what you need to know about OSPF point-to-point:

* Automatic neighbor discovery so no need to configure OSPF neighbors yourself.
* No DR/BDR election since OSPF sees the network as a collection of point-to-point links.
* Normally uses for point-to-point sub-interfaces with an IP subnet per link.
* Can also be used with multiple PVCs using only one subnet.

Let me show you the configuration of the Hub router:

<pre><code><strong>Hub(config)#interface serial 0/0
</strong><strong>Hub(config-if)#encapsulation frame-relay 
</strong><strong>Hub(config-if)#exit
</strong><strong>Hub(config)#interface serial 0/0.102 point-to-point 
</strong><strong>Hub(config-subif)#ip address 192.168.12.1 255.255.255.0
</strong><strong>Hub(config-subif)#frame-relay interface-dlci 102
</strong><strong>Hub(config-subif)#exit
</strong><strong>Hub(config)#interface serial 0/0.103 point-to-point
</strong><strong>Hub(config-subif)#ip address 192.168.13.1 255.255.255.0
</strong><strong>Hub(config-subif)#frame-relay interface-dlci 103
</strong></code></pre>

I am using two sub-interfaces and assigning the correct DLCI number to each sub-interface. Now let’s configure the spoke routers:

<pre><code><strong>Spoke1(config)#interface serial 0/0
</strong><strong>Spoke1(config-if)#encapsulation frame-relay 
</strong><strong>Spoke1(config-if)#interface serial 0/0.201 point-to-point
</strong><strong>Spoke1(config-subif)#ip address 192.168.12.2 255.255.255.0
</strong><strong>Spoke1(config-if)#frame-relay interface-dlci 201
</strong></code></pre>

<pre><code><strong>Spoke2(config)#interface serial 0/0
</strong><strong>Spoke2(config-if)#encapsulation frame-relay 
</strong><strong>Spoke2(config-if)#interface serial 0/0.301 point-to-point
</strong><strong>Spoke2(config-subif)#ip address 192.168.13.3 255.255.255.0
</strong><strong>Spoke2(config-if)#frame-relay interface-dlci 301
</strong></code></pre>

Above you see a sub-interface for each spoke router with the correct DLCI number.

{% hint style="info" %}
Keep in mind that the physical interface for frame-relay is always point-to-multipoint. If the technical requirements state that you need to use a point-to-point interface then you should create a sub-interface.
{% endhint %}

Let’s enable OSPF for all routers:

<pre><code><strong>Hub(config)#router ospf 1
</strong><strong>Hub(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong><strong>Hub(config-router)#network 192.168.13.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>Spoke1(config)#router ospf 1
</strong><strong>Spoke1(config-router)#network 192.168.12.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>Spoke2(config)#router ospf 1
</strong><strong>Spoke2(config-router)#network 192.168.13.0 0.0.0.255 area 0
</strong></code></pre>

Configure OSPF for all subnets, nothing special here…let’s see if we have any OSPF neighbors:

<pre><code><strong>Hub#show ip ospf neighbor 
</strong>
Neighbor ID     Pri   State        Dead Time   Address         Interface
192.168.13.3     0   FULL/  -     00:00:34    192.168.13.3    Serial0/0.103
192.168.12.2     0   FULL/  -     00:00:36    192.168.12.2    Serial0/0.102
</code></pre>

As you can see we have OSPF neighbors…mission accomplished!

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="Hub" %}
```
hostname Hub
!
ip cef
!
interface Serial0/0
 no ip address
 encapsulation frame-relay
!
interface Serial0/0.102 point-to-point
 ip address 192.168.12.1 255.255.255.0
 frame-relay interface-dlci 102
!
interface Serial0/0.103 point-to-point
 ip address 192.168.13.1 255.255.255.0
 frame-relay interface-dlci 103
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.13.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="Spoke1" %}
```
hostname Spoke1
!
ip cef
!
interface Serial0/0.201 point-to-point
 ip address 192.168.12.2 255.255.255.0
 frame-relay interface-dlci 201
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="Spoke2" %}
```
hostname Spoke2
!
ip cef
!
interface Serial0/0.301 point-to-point
 ip address 192.168.13.3 255.255.255.0
 frame-relay interface-dlci 301
!
router ospf 1
 network 192.168.13.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

If you have any questions feel free to ask them.
