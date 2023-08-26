# EIGRP Static Neighbor

EIGRP by default uses multicast for neighbor discovery but it also allows you to configure EIGRP neighbors statically. Once you do this, EIGRP will **only use unicast** and disables EIGRP multicast on the selected interface.

This could be useful in certain scenarios where multicast is not supported or when you want to reduce the overhead of multicast traffic. Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/eigrp-frame-relay-multicast-replicated-pvcs.png" alt=""><figcaption></figcaption></figure>

Above we have a frame-relay hub and spoke network. The hub and spoke1 routers are the only two routers that are running EIGRP. When the hub router sends an EIGRP multicast packet, it will be replicated on all PVCs. All 4 spoke routers will receive this multicast traffic even though only spoke1 is interested in it.

In a scenario like this, it would make sense to configure the EIGRP neighbor statically so that multicast won’t be used.

Let’s see how we can configure EIGRP static neighbors…

## Configuration

For this demonstration I’ll use the following two routers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/06/eigrp-frame-relay-two-routers.png" alt=""><figcaption></figcaption></figure>

R1 and R2 are connected through frame-relay. Here’s the configuration of the interfaces:

<pre><code><strong>R1#show run | begin Serial0/0
</strong>interface Serial0/0
 ip address 192.168.12.1 255.255.255.0
 encapsulation frame-relay
 clock rate 2000000
 frame-relay map ip 192.168.12.2 102
 no frame-relay inverse-arp
</code></pre>

<pre><code><strong>R2#show run | begin Serial0/0 
</strong>interface Serial0/0
 ip address 192.168.12.2 255.255.255.0
 encapsulation frame-relay
 clock rate 2000000
 frame-relay map ip 192.168.12.1 201
 no frame-relay inverse-arp
</code></pre>

Above you can see that frame-relay Inverse ARP has been disabled, two static frame-relay maps are used for our mappings. This means that we are unable to send any broadcast or multicast traffic through this PVC. You can also verify this with the following command:

<pre><code><strong>R1#show frame-relay map
</strong>Serial0/0 (up): ip 192.168.12.2 dlci 102(0x66,0x1860), static,
              CISCO, status defined, active
</code></pre>

<pre><code><strong>R2#show frame-relay map 
</strong>Serial0/0 (up): ip 192.168.12.1 dlci 201(0xC9,0x3090), static,
              CISCO, status defined, active
</code></pre>

As you can see the frame-relay mappings are there but the broadcast keyword is missing. Let’s configure EIGRP to use static neighbors:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#network 192.168.12.0
</strong><strong>R1(config-router)#neighbor 192.168.12.2 Serial 0/0 
</strong></code></pre>

<pre><code><strong>R2(config)#router eigrp 12
</strong><strong>R2(config-router)#network 192.168.12.0
</strong><strong>R2(config-router)#neighbor 192.168.12.1 Serial 0/0
</strong></code></pre>

You only have to use the neighbor command to specify the remote neighbor and the interface to reach it. After a few seconds the neighbor adjacency will appear:

```
R1#
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 12: Neighbor 192.168.12.2 (Serial0/0) is up: new adjacency
```

```
R2#
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 12: Neighbor 192.168.12.1 (Serial0/0) is up: new adjacency
```

You you can also verify your static neighbors with the **show ip eigrp neighbors** command:

<pre><code><strong>R1#show ip eigrp neighbors detail 
</strong>IP-EIGRP neighbors for process 12
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
0   192.168.12.2            Se0/0            154 00:01:16   34   204  0  3
   Static neighbor
   Version 12.4/1.2, Retrans: 0, Retries: 0
</code></pre>

Make sure you add the **detail** parameter or it will only show the EIGRP neighbors, not if they are static neighbors or not.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface Serial0/0
 ip address 192.168.12.1 255.255.255.0
 encapsulation frame-relay
 clock rate 2000000
 frame-relay map ip 192.168.12.2 102
 no frame-relay inverse-arp
!
router eigrp 12
 network 192.168.12.0
 neighbor 192.168.12.2 Serial 0/0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface Serial0/0
 ip address 192.168.12.2 255.255.255.0
 encapsulation frame-relay
 clock rate 2000000
 frame-relay map ip 192.168.12.1 201
 no frame-relay inverse-arp
!
router eigrp 12
 network 192.168.12.0
 neighbor 192.168.12.1 Serial 0/0
!
end
```
{% endtab %}
{% endtabs %}

\
That’s all there is to it. I hope this example has been useful to understand EIGRP static neighbors. Any questions? Feel free to leave a comment!
