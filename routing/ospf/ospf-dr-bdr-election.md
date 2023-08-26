# OSPF DR/BDR Election

OSPF uses a DR (Designated Router) and BDR (Backup Designated Router) on each multi-access network. A multi-access network is a segment where we have more than two routers. OSPF figures this out by looking at the interface type. For example, an Ethernet interface is considered a multi-access network, and a serial interface is considered a point-to-point interface.

Most CCNA students think that this DR/BDR election is done per area but this is **incorrect**. I’ll show you how the election is done and how you can influence it. This is the topology we’ll use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-3-routers-multi-access.png" alt=""><figcaption></figcaption></figure>

Here’s an example of a network with 3 OSPF routers on a FastEthernet network. They are connected to the same switch (multi-access network) so there will be a DR/BDR election. OSPF has been configured, so all routers have become OSPF neighbors. Let’s take a look:

<pre><code><strong>R1#show ip ospf neighbor
</strong>
Neighbor ID     Pri   State       Dead Time   Address       Interface
192.168.123.2   1   FULL/BDR      00:00:32    192.168.123.2 FastEthernet0/0
192.168.123.3   1   FULL/DR       00:00:31    192.168.123.3 FastEthernet0/0
</code></pre>

From R1’s perspective, R2 is the BDR, and R3 is the DR.

<pre><code><strong>R3#show ip ospf neighbor
</strong>
Neighbor ID     Pri   State       Dead Time   Address         Interface
192.168.123.1   1   FULL/DROTHER  00:00:36    192.168.123.1 FastEthernet0/0
192.168.123.2   1   FULL/BDR      00:00:39    192.168.123.2 FastEthernet0/0
</code></pre>

When a router is not the DR or BDR, it’s called a **DROTHER**. Here we can see that R1 is a DROTHER.

<pre><code><strong>R2#show ip ospf neighbor
</strong>
Neighbor ID     Pri   State       Dead Time   Address         Interface
192.168.123.1   1   FULL/DROTHER  00:00:31    192.168.123.1 FastEthernet0/0
192.168.123.3   1   FULL/DR       00:00:32    192.168.123.3 FastEthernet0/0
</code></pre>

And R2 (the BDR) sees the DR and DROTHER.

Of course, we can change which router becomes the DR/BDR by playing with the priority. Let’s turn R1 in the DR:

<pre><code><strong>R1(config)#interface fastEthernet 0/0
</strong><strong>R1(config-if)#ip ospf priority 200
</strong></code></pre>

You change the priority if you like by using the ip ospf priority command:

* The default priority is 1.
* A priority of 0 means you will never be elected as DR or BDR.
* You need to use **`clear ip ospf process`** before this change takes effect.

<pre><code><strong>R1#show ip ospf neighbor 
</strong>
Neighbor ID     Pri   State       Dead Time   Address         Interface
192.168.123.2   1   FULL/BDR      00:00:31    192.168.123.2 FastEthernet0/0
192.168.123.3   1   FULL/DR       00:00:32    192.168.123.3 FastEthernet0/0
</code></pre>

As you can see, R3 is still the DR, we need to reset the OSPF neighbor adjacencies so that we’ll elect the new DR and BDR.

<pre><code><strong>R3#clear ip ospf process 
</strong>Reset ALL OSPF processes? [no]: yes
</code></pre>

<pre><code><strong>R2#clear ip ospf process 
</strong>Reset ALL OSPF processes? [no]: yes
</code></pre>

I’ll reset all the OPSF neighbor adjacencies.

<pre><code><strong>R1#show ip ospf neighbor 
</strong>
Neighbor ID     Pri   State       Dead Time   Address         Interface
192.168.123.2   1   FULL/DROTHER  00:00:36    192.168.123.2 FastEthernet0/0
192.168.123.3   1   FULL/BDR      00:00:30    192.168.123.3 FastEthernet0/0
</code></pre>

Now you can see R1 is the DR because the other routers are DROTHER and BDR.

<pre><code><strong>R3#show ip ospf neighbor
</strong>Neighbor ID     Pri  State        Dead Time   Address         Interface
192.168.123.1   200  FULL/DR      00:00:30    192.168.123.1 FastEthernet0/0
192.168.123.2   1    FULL/DROTHER 00:00:31    192.168.123.2 FastEthernet0/0
</code></pre>

Or we can confirm it from R3. You’ll see that R1 is the DR and that the priority is 200.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip cef
!
interface FastEthernet0/0
 ip address 192.168.123.1 255.255.255.0
 ip ospf priority 200
!
router ospf 1
 network 192.168.123.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip cef
!
interface FastEthernet0/0
 ip address 192.168.123.2 255.255.255.0
!
router ospf 1
 network 192.168.123.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ip cef
!
interface FastEthernet0/0
 ip address 192.168.123.3 255.255.255.0
!
router ospf 1
 network 192.168.123.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

Something you need to be aware of is that the DR/BDR election is per multi-access segment…not per area!). Let me give you an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-r1-r2-r3-two-broadcast-domains.png" alt=""><figcaption></figcaption></figure>

In the example above, we have two multi-access segments. Between R2 and R1, and between R2 and R3. For each segment, there will be a DR/BDR election.

<pre><code><strong>R2#show ip ospf neighbor 
</strong>
Neighbor ID     Pri   State     Dead Time   Address         Interface
192.168.23.3    200   FULL/DR   00:00:36    192.168.23.3    FastEthernet1/0
192.168.12.1    200   FULL/DR   00:00:37    192.168.12.1    FastEthernet0/0
</code></pre>

In the example above, you can see that:

* R1 is the DR for the 192.168.12.0/24 segment.
* R3 is the DR for the 192.168.23.0/24 segment.

This also means that R2 is the BDR for the 192.168.12.0/24 and the BDR for the 192.168.23.0/24 segment.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip cef
!
interface FastEthernet0/0
 ip address 192.168.12.1 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip cef
!
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
interface FastEthernet1/0
 ip address 192.168.23.2 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
 network 192.168.13.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ip cef
!
interface FastEthernet0/0
 ip address 192.168.23.3 255.255.255.0
!
router ospf 1
 network 192.168.13.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

Last but not least, let me show you an example where we **don’t** have a DR/BDR election:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/ospf-point-to-point.png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>R1#show ip ospf neighbor
</strong>
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.12.2      0   FULL/  -        00:00:36    192.168.12.2    Serial0/0
</code></pre>

<pre><code><strong>R2#show ip ospf neighbor
</strong>
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.12.1      0   FULL/  -        00:00:34    192.168.12.1    Serial0/0
</code></pre>

Here’s an example of a point-to-point link running HDLC. You can see that we have a neighbor, but we didn’t do an election for DR or BDR. It makes sense because there is always only one router on the other side.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ip cef
!
interface Serial0/0
 ip address 192.168.12.1 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip cef
!
interface Serial0/0
 ip address 192.168.12.2 255.255.255.0
!
router ospf 1
 network 192.168.12.0 0.0.0.255 area 0
!
end
```
{% endtab %}
{% endtabs %}

That’s all I wanted to show you for now. I hope you enjoyed reading this lesson. If you have any questions, please leave a comment!
