# PAT Configuration

I have covered the configuration of static NAT and dynamic NAT in previous lessons, now it’s time for PAT. This is the topology we’ll use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/nat-2-hosts-inside-outside-1.png" alt=""><figcaption></figcaption></figure>

Let’s prepare the hosts. I am using normal Cisco routers with `ip routing` disabled to turn them into dumb hosts:

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

So far so good. Let’s create an access-list that matches both hosts:

<pre><code><strong>NAT(config)#access-list 1 permit 192.168.123.0 0.0.0.255
</strong></code></pre>

And finally, we’ll configure PAT:

<pre><code><strong>NAT(config)#ip nat inside source list 1 interface fastEthernet 1/0 overload
</strong></code></pre>

I select access-list 1 as my inside source, and I will translate them to the IP address on FastEthernet 1/0. The big magic keyword here is `overload`. If you add this, we will enable PAT!

Let’s give it a test run, shall we?

To take a closer look at the port number, I won’t use ping, but we’ll connect to TCP port 80 of the webserver (Web1):

<pre><code><strong>Web1(config)#ip http server
</strong></code></pre>

First, we’ll enable the web server.

We can use telnet to connect to port 80:

<pre><code><strong>Host1#telnet 192.168.23.3 80
</strong>Trying 192.168.23.3, 80 ... Open
</code></pre>

<pre><code><strong>Host2#telnet 192.168.23.3 80
</strong>Trying 192.168.23.3, 80 ... Open
</code></pre>

As you see, it says “open,” which means that we successfully connected to port 80.

Let’s see what the NAT/PAT table looks like now:

<pre><code><strong>NAT#show ip nat translations  
</strong>Pro Inside global      Inside local       Outside local      Outside global
tcp 192.168.23.2:46369 192.168.123.1:46369 192.168.23.3:80  192.168.23.3:80
tcp 192.168.23.2:50669 192.168.123.2:50669 192.168.23.3:80  192.168.23.3:80
</code></pre>

Above, you see that it keeps track of the port number and that both hosts are translated to IP address 192.168.23.2. Mission accomplished!

{% hint style="info" %}
Telnet is a great command to connect to different TCP ports. You can use it to test access lists or connectivity…or in my example, to play with NAT/PAT.
{% endhint %}

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
access-list 1 permit 192.168.123.0 0.0.0.255
ip nat inside source list 1 interface fastEthernet 1/0 overload
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
ip http server
!
end
```
{% endtab %}
{% endtabs %}

That’s it! You have now learned how to configure PAT on your Cisco IOS router. If you enjoyed this lesson or have more questions, please comment!
