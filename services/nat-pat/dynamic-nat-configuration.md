# Dynamic NAT Configuration

It’s time to configure dynamic NAT where we use a pool of IP addresses for translation. I’ll use a fairly simple topology with two hosts and one router that will perform NAT:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/cisco-dynamic-nat-two-hosts..png" alt=""><figcaption></figcaption></figure>

This time we have 2 host routers on the left side, and I’m using another subnet. Let’s prepare the host routers:

<pre><code><strong>Host1(config)#no ip routing
</strong><strong>Host1(config)#default gateway 192.168.123.3
</strong></code></pre>

<pre><code><strong>Host2(config)#no ip routing
</strong><strong>Host2(config)#ip default-gateway 192.168.123.3
</strong></code></pre>

The next step is to configure NAT:

<pre><code><strong>NAT(config)#interface fastEthernet 0/0
</strong><strong>NAT(config-if)#ip nat inside 
</strong></code></pre>

<pre><code><strong>NAT(config)#interface fastEthernet 1/0
</strong><strong>NAT(config-if)#ip nat outside
</strong></code></pre>

First, we’ll configure the correct inside and outside interfaces. Now I will create a pool with IP addresses that we can use for the translation:

<pre><code><strong>NAT(config)#ip nat pool MYPOOL 192.168.23.10 192.168.23.20 prefix-length 24
</strong></code></pre>

The `ip nat pool` command lets us create a pool. I’m calling mine “MYPOOL,” and I’m using IP address 192.168.23.10 up to 192.168.23.20. We can now select the hosts that we want to translate:

<pre><code><strong>NAT(config)#access-list 1 permit 192.168.123.0 0.0.0.255
</strong></code></pre>

The access-list above matches network 192.168.123.0 /24. That’s where host1 and host2 are located. The last step is to put the access-list and pool together:

<pre><code><strong>NAT(config)#ip nat inside source list 1 pool MYPOOL
</strong></code></pre>

The command above selects access-list 1 as the source, and we will translate it to the pool called “MYPOOL.” This ensures that host1 and host2 are translated to an IP address from our pool. Now let’s verify our configuration!

<pre><code><strong>Host1#ping 192.168.23.3
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.23.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/12/28 ms
</code></pre>

Host1 can ping the web server. Now let’s take a look at our NAT router:

<pre><code><strong>NAT#show ip nat translations  
</strong>Pro Inside global      Inside local       Outside local      Outside global
icmp 192.168.23.10:3   192.168.123.1:3    192.168.23.3:3     192.168.23.3:3
--- 192.168.23.10      192.168.123.1      ---                ---
</code></pre>

As you can see above, host1 has been translated to IP address 192.168.23.10. Now let’s send some traffic from host2 to see the difference in our NAT table…

<pre><code><strong>Host2#ping 192.168.23.3
</strong>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.23.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/9/16 ms
</code></pre>

<pre><code><strong>NAT#show ip nat translations 
</strong>Pro Inside global      Inside local       Outside local      Outside global
icmp 192.168.23.10:4   192.168.123.1:4    192.168.23.3:4     192.168.23.3:4
--- 192.168.23.10      192.168.123.1      ---                ---
icmp 192.168.23.11:2   192.168.123.2:2    192.168.23.3:2     192.168.23.3:2
--- 192.168.23.11      192.168.123.2      ---                ---
</code></pre>

And as you can see, host2 has been translated to IP address 192.168.2.11. Excellent, our dynamic NAT is working! In case you are wondering…what do the inside global, inside local, outside local, and outside global addresses mean? Let me explain to you:

* Inside global is the IP address on the outside interface of your router performing NAT.
* Inside local is the IP address of one of your inside hosts that is translated with NAT.
* Outside local is the IP address of the device you are trying to reach, in our example, the webserver (Web1).
* Outside global is also the IP address of the device you are trying to reach, in our example, the webserver (Web1).

Why are the outside local and outside global IP addresses the same? With NAT, it’s possible to translate more than just from “inside” to “outside.” It’s possible to create an entry in our NAT router that whenever one of the hosts sends a ping to an IP address (let’s say 5.5.5.5) that it will be forwarded to Web1. In this example, the “outside webserver” is “locally” seen by our hosts as 5.5.5.5, not 192.168.23.3.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="Host1" %}
```
hostname Host1
!
interface FastEthernet0/0
 ip address 192.168.123.1 255.255.255.0
!
no ip routing
ip default-gateway 192.168.123.3
!
end
```
{% endtab %}

{% tab title="Host2" %}
```
hostname Host2
!
interface FastEthernet0/0
 ip address 192.168.123.2 255.255.255.0
!
no ip routing
ip default-gateway 192.168.123.3
!
end
```
{% endtab %}

{% tab title="NAT" %}
```
hostname NAT
!
interface FastEthernet0/0
 ip address 192.168.123.3 255.255.255.0
 ip nat inside 
!
interface FastEthernet1/0
 ip address 192.168.23.2 255.255.255.0
 ip nat outside
!
ip nat pool MYPOOL 192.168.23.10 192.168.23.20 prefix-length 24
access-list 1 permit 192.168.123.0 0.0.0.255
ip nat inside source list 1 pool MYPOOL
!
end
```
{% endtab %}

{% tab title="Web1" %}
```
hostname Web1
!
interface FastEthernet0/0
 ip address 192.168.23.3 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

That’s the end of this Dynamic NAT lesson. I hope this has been helpful to you. If you have any questions feel free to ask!
