# IP NAT Inside Source vs Outside Source

On Cisco IOS routers we can use the `ip nat inside source`and `ip nat outside source` commands. Most of us are familiar with the `ip nat inside source` command because we often use it to translate private IP addressses on our LAN to a public IP address we received from our ISP.

What about the `ip nat outside source` command? Does it work in the same way as `ip nat inside source`?

This is the difference between the two commands:

#### **ip nat inside source**:

* Translates the source IP address of packets that travel from inside to outside.
* Translates the destination IP address of packets that travel from outside to inside.

#### **ip nat outside source**:

* Translates the source IP address of packets that travel from outside to inside.
* Translates the destination IP address of packets that travel from inside to outside.

## Configuration

Let’s look at these two commands in action. I use the following topology to demonstrate this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/11/r1-h1-h2-nat-inside-outside-source-topology-1.png" alt=""><figcaption></figcaption></figure>

IP routing is disabled on H1 and H2, they use R1 as their default gateway.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the startup configuration of each device.
{% endtab %}

{% tab title="H1" %}
```
```
{% endtab %}

{% tab title="H2" %}
```
```
{% endtab %}

{% tab title="R1" %}
```
```
{% endtab %}
{% endtabs %}

Let’s enable NAT debugging on R1 so we can see everything in action:

<pre><code><strong>R1#debug ip nat 
</strong>IP NAT debugging is on
</code></pre>

### IP NAT inside source

Let’s start with `ip nat inside source`, the command we are most familiar with. I’ll configure an entry that translates 192.168.1.1 to 192.168.2.200:

<pre><code><strong>R1(config)#ip nat inside source static 192.168.1.1 192.168.2.200
</strong></code></pre>

Let’s send a ping from H1 to 192.168.2.2:

<pre><code><strong>H1#ping 192.168.2.2 repeat 1 
</strong>Type escape sequence to abort. 
Sending 1, 100-byte ICMP Echos to 192.168.2.2, timeout is 2 seconds: 
! 
Success rate is 100 percent (1/1), round-trip min/avg/max = 4/4/4 ms
</code></pre>

R1 produces the following debug output:

```
R1# 
NAT*: s=192.168.1.1->192.168.2.200, d=192.168.2.2 [3] 
NAT*: s=192.168.2.2, d=192.168.2.200->192.168.1.1 [3]
```

* The source IP address 192.168.1.1 is translated to 192.168.2.200 when the IP packet travels from the inside to the outside.
* The destination IP address 192.168.2.200 is translated to 192.168.1.1 when the return IP packet travels from the outside to inside.

We can also try a ping from H2. Let’s see what happens when we ping 192.168.2.200:

<pre><code><strong>H2#ping 192.168.2.200 repeat 1 
</strong>Type escape sequence to abort. 
Sending 1, 100-byte ICMP Echos to 192.168.2.200, timeout is 2 seconds: 
! 
Success rate is 100 percent (1/1), round-trip min/avg/max = 5/5/5 ms
</code></pre>

R1 produces the following debug output:

```
R1# 
NAT*: s=192.168.2.2, d=192.168.2.200->192.168.1.1 [8] 
NAT*: s=192.168.1.1->192.168.2.200, d=192.168.2.2 [8]
```

* The destination IP address is translated from 192.168.2.200 to 192.168.1.1 when the IP packet travels from the outside to the inside.
* The source IP address is translated from 192.168.1.1 to 192.168.2.200 when the return IP packet travels from the inside to the outside.

Can I ping the 192.168.1.1 IP address from H2? Let’s find out:

<pre><code><strong>H2#ping 192.168.1.1 repeat 1 
</strong>Type escape sequence to abort. 
Sending 1, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds: 
! 
Success rate is 100 percent (1/1), round-trip min/avg/max = 6/6/6 ms
</code></pre>

This is what we see on R1:

```
R1# 
NAT*: s=192.168.1.1->192.168.2.200, d=192.168.2.2 [6]
```

The source IP address 192.168.1.1 is translated to 192.168.2.00 when it travels from the inside to the outside.

### IP NAT outside source

Let’s find out how the `ip nat outside source` command works. I’ll use the following command:

<pre><code><strong>R1(config)#R1(config)#ip nat outside source static 192.168.2.2 192.168.2.200
</strong></code></pre>

This translates source IP address 192.168.2.2 to 192.168.2.200 when the IP packet travels from the outside to the inside.

Let’s try a ping from H2 to 192.168.1.1:

<pre><code><strong>H2#ping 192.168.1.1 repeat 1 
</strong>Type escape sequence to abort. 
Sending 1, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds: 
! 
Success rate is 100 percent (1/1), round-trip min/avg/max = 8/8/8 ms
</code></pre>

We see the following NAT translations on R1:

```
R1# 
NAT: s=192.168.2.2->192.168.2.200, d=192.168.1.1 [9] 
NAT: s=192.168.1.1, d=192.168.2.200->192.168.2.2 [9]
```

* Source IP address 192.168.2.2 is translated to 192.168.2.200 when the IP packet travels from the outside to the inside.
* Destination IP address 192.168.2.200 is translated to 192.168.2.2 when the return IP packet travels from the inside to the outside.

What about a ping from H1 to 192.168.2.200?

<pre><code><strong>H1#ping 192.168.2.200 repeat 1 
</strong>Type escape sequence to abort. 
Sending 1, 100-byte ICMP Echos to 192.168.2.200, timeout is 2 seconds: 
! 
Success rate is 100 percent (1/1), round-trip min/avg/max = 6/6/6 ms
</code></pre>

Here’s the debug on R1:

```
R1# 
NAT: s=192.168.1.1, d=192.168.2.200->192.168.2.2 [11] 
NAT*: s=192.168.2.2->192.168.2.200, d=192.168.1.1 [11]
```

* Destination IP address 192.168.2.200 is translated to 192.168.2.2 when the IP packet travels from the inside to the outside.
* Source IP address 192.168.2.2 is translated to 192.168.2.200 when the return IP packet travels from the outside to the inside.

## Conclusion

You have now learned the difference between the `ip nat inside source` and `ip nat outside source` commands:

* ip nat inside source:
  * translates the source IP address when a packet travels from the inside to the outside.
  * translates the destination IP address when a packet travels from the outside to the inside.
* ip nat outside source:
  * translates the source IP address when a packet travels from the outside to the inside.
  * translates the destination IP address when a packet travels from the inside to the outside.

I hope you enjoyed this lesson. If you have any questions feel free to leave a comment!
