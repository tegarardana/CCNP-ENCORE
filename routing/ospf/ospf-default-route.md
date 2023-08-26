# OSPF Default Route

With OSPF, it is no problem to configure a default route. There are a couple of options if you want to do this. Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/r1-r2-ospf-area-0.png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>R1(config)#router ospf 1  
</strong><strong>R1(config-router)#default-information originate ?
</strong>  always       Always advertise default route
  metric       OSPF default metric
  metric-type  OSPF metric type for default routes
  route-map    Route-map reference
  &#x3C;cr>
</code></pre>

There are a number of things. We can change the metric or metric type, but the most important thing most people forget is the **`always`** keyword.

If you use the `default-information originate` command, you can advertise a default route in OSPF. OSPF won’t advertise a default route if you don’t already **have it in your routing table**. If you add the `always` keyword, it will advertise the default route even if you don’t have it in the routing table. Once you have advertised the default route, it will look like this on other routers:

<pre><code><strong>R2#show ip ospf database | begin Type-5
</strong><strong>		Type-5 AS External Link States
</strong>
Link ID         ADV Router      Age         Seq#       Checksum Tag
0.0.0.0         172.16.3.1      59          0x80000001 0x008D64 1
</code></pre>

<pre><code><strong>R2#show ip route ospf 
</strong>O*E2 0.0.0.0/0 [110/1] via 192.168.12.1, 00:00:24, FastEthernet0/0
</code></pre>

It will show up as an LSA type 5 external route.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet0/1
 ip address 192.168.12.1 255.255.255.0
!
router ospf 1
 network 192.168.12.0
 default-information originate always
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface FastEthernet0/1
 ip address 192.168.12.2 255.255.255.0
!
router ospf 1
 network 192.168.12.0
!
end
```
{% endtab %}
{% endtabs %}

I hope this is helpful to you. If you have any questions leave a comment.
