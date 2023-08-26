# OSPF Router ID

Each OSPF router selects a router ID (RID) that has to be unique on your network. OSPF stores the topology of the network in its LSDB (Link State Database) and each router is identified with its unique router ID , if you have duplicate router IDs then you will run into reachability issues.

Because of this, two OSPF routers with the same router ID will not become neighbors but you could still have duplicated router IDs in the network with routers that are not directly connected to each other.

OSPF uses the following criteria to select the router ID:

1. Manual configuration of the router ID.
2. Highest IP address on a loopback interface.
3. Highest IP address on a non-loopback interface.

Let’s see this in action. I will use the following router for this demonstration:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/07/cisco-router-ospf-4-interfaces.png" alt=""><figcaption></figcaption></figure>

There are two physical interfaces and two loopback interfaces. All interfaces are active:

<pre><code><strong>R1#show ip interface brief
</strong>Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.1.1     YES manual up                    up
FastEthernet0/1            192.168.11.1    YES manual up                    up
Loopback0                  1.1.1.1         YES manual up                    up
Loopback1                  11.11.11.11     YES manual up                    up
</code></pre>

Let’s start an OSPF process:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#exit
</strong></code></pre>

Now we can check what router ID it selected:

<pre><code><strong>R1#show ip protocols | include Router ID
</strong>  Router ID 11.11.11.11
</code></pre>

It selected 11.11.11.11 which is the highest IP address on our loopback interfaces. Let’s get rid of the loopbacks now:

<pre><code><strong>R1(config)#no interface loopback 0
</strong><strong>R1(config)#no interface loopback 1
</strong></code></pre>

Take a look again at the router ID:

<pre><code><strong>R1#show ip protocols | include Router ID
</strong>  Router ID 11.11.11.11
</code></pre>

It’s still the same, this is because the router ID selection is only done once. You have to reset the OSPF process before it will select another one:

<pre><code><strong>R1#clear ip ospf process
</strong>Reset ALL OSPF processes? [no]: yes
</code></pre>

Let’s see if this makes any difference:

<pre><code><strong>R1#show ip protocols | include Router ID
</strong>  Router ID 192.168.11.1
</code></pre>

There we go, the router ID is now the highest IP address of our physical interfaces. If you want we can manually set the router ID. This will overrule everything:

<pre><code><strong>R1(config)#router ospf 1
</strong><strong>R1(config-router)#router-id 111.111.111.111
</strong></code></pre>

Use the **router-id** command for this. Let’s verify our work:

<pre><code><strong>R1#show ip protocols | include Router ID
</strong>  Router ID 111.111.111.111
</code></pre>

There we go, our new router ID is visible.

{% hint style="info" %}
When your router doesn’t have any neighbors then OSPF will immediately change the router ID (if you used the router-id command). When you do have neighbors you will have to reset the OSPF process.
{% endhint %}

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of R1.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet0/1
 ip address 192.168.11.1 255.255.255.0
!
interface FastEthernet0/0
 ip address 192.168.1.1 255.255.255.0
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.0
!
interface Loopback1
 ip address 11.11.11.11 255.255.255.0
!
router ospf 1
 router id 111.111.111.111
!
end
```
{% endtab %}
{% endtabs %}

That’s all there is to it, I hope this example has been useful! Any questions? Just leave a comment.
