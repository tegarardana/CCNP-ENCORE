# OSPF Network Type Non-Broadcast

The different OSPF network types are a pain to understand for most students. I’m going to cover all of them in a series of lessons and in this one, we’ll take a look at the OSPF non-broadcast network type. We will be using a point-to-multipoint frame-relay network for the demonstration:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-network-type-topology.png" alt=""><figcaption></figcaption></figure>

If you select the non-broadcast network type, then OSPF will assume you are running a **non-broadcast** multi-access **(NBMA)** network. Here are a couple of key things to remember here:

* Multi-access means we have to select a DR and BDR.
* Non-broadcast means that OSPF expects us to configure neighbors ourselves.

Now look closely at my frame-relay network in the picture above and assume we will run the non-broadcast network type here. Is my network multi-access? Interesting question, right?

Is there any connectivity between routers Spoke1 and Spoke2? You can see I only have 2 PVCs, and there is no connection between Spoke1 and Spoke2. This is where things can get funky! If we select the non-broadcast network type we are telling OSPF our network is multi-access, but in reality, it’s not…there is only connectivity between the Hub router and the Spoke routers, not between the 2 spoke routers.

When I explained the DR/BDR to you, I told you that we only have a full adjacency with the DR/BDR and not with all other routers. What do you think will happen if Spoke1 is elected as the DR?

Since Spoke2 can’t reach Spoke1, it can never set up a full OSPF neighbor adjacency, and we’ll run into connectivity issues. How do we solve this? We have to make sure the Hub router becomes the DR, and Spoke1 or Spoke2 will never become DR or BDR!

The Hub router can be reached by Spoke1 and Spoke2, so if it’s the DR, they can do the full OSPF neighbor adjacency, exchange routing information, and life is all good. Let’s take a look at the configuration:

<pre><code><strong>Hub(config)#interface serial 0/0
</strong><strong>Hub(config-if)#ip address 192.168.123.1 255.255.255.0
</strong><strong>Hub(config-if)#encapsulation frame-relay
</strong><strong>Hub(config-if)#ip ospf network non-broadcast
</strong><strong>Hub(config-if)#exit
</strong><strong>Hub(config)#router ospf 1
</strong><strong>Hub(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong><strong>Hub(config-router)#neighbor 192.168.123.2
</strong><strong>Hub(config-router)#neighbor 192.168.123.3
</strong></code></pre>

Here is the configuration of the Hub router. I used the `ip ospf network non-broadcast` command to change the OSPF network type. I also had to specify the neighbors, so OSPF switches to unicast.

<pre><code><strong>Spoke1(config)#interface serial 0/0
</strong><strong>Spoke1(config-if)#ip address 192.168.123.2 255.255.255.0
</strong><strong>Spoke1(config-if)#encapsulation frame-relay 
</strong><strong>Spoke1(config-if)#ip ospf network non-broadcast 
</strong><strong>Spoke1(config-if)#ip ospf priority 0
</strong><strong>Spoke1(config-if)#exit
</strong><strong>Spoke1(config)#router ospf 1
</strong><strong>Spoke1(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

<pre><code><strong>Spoke2(config)#interface serial 0/0
</strong><strong>Spoke2(config-if)#ip address 192.168.123.3 255.255.255.0
</strong><strong>Spoke2(config-if)#encapsulation frame-relay 
</strong><strong>Spoke2(config-if)#ip ospf network non-broadcast 
</strong><strong>Spoke2(config-if)#ip ospf priority 0
</strong><strong>Spoke2(config-if)#exit
</strong><strong>Spoke2(config)#router ospf 1
</strong><strong>Spoke2(config-router)#network 192.168.123.0 0.0.0.255 area 0
</strong></code></pre>

Here is the configuration of the spoke routers. I changed the OSPF network type as well, but there’s one little extra. See the `ip ospf priority 0`? That’s how you ensure these routers will never become a DR or BDR.

{% hint style="info" %}
Make sure you have a frame-relay map statement with the broadcast keyword, or you won’t be able to send multicast on your frame-relay network. By default Inverse ARP is enabled and will do this for you…if you don’t have inverse ARP, make sure you add it!
{% endhint %}

<pre><code><strong>Hub#show ip ospf neighbor 
</strong>
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.123.2     0   FULL/DROTHER    00:00:30    192.168.123.2   Serial0/0
192.168.123.3     0   FULL/DROTHER    00:00:35    192.168.123.3   Serial0/0
</code></pre>

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
 ip ospf network non-broadcast
 clock rate 2000000
!
router ospf 1
 network 192.168.123.0 0.0.0.255 area 0
 neighbor 192.168.123.2
 neighbor 192.168.123.3
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
 ip ospf network non-broadcast
 ip ospf priority 0
 clock rate 2000000
!
router ospf 1
 log-adjacency-changes
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
 ip ospf network non-broadcast
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
