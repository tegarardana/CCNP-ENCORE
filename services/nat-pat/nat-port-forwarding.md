# NAT Port Forwarding

NAT port forwarding is typically used to allow remote hosts to connect to a host or server on our private LAN. A host on the outside (for example on the Internet) will connect to the outside IP address of a router that is configured for NAT. This NAT router will forward traffic to host on the inside. Here’s an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/05/nat-port-forwarding-inside-outside.png" alt=""><figcaption></figcaption></figure>

Above we have three routers, we’ll use these to demonstrate NAT port forwarding. Imagine R1 is a HTTP server on our LAN and R3 is some host on the Internet that wants to reach our HTTP server. R2 will make sure that the HTTP server is reachable on an IP address on the outside. Let’s take a look at the configuration…

## Configuration

First we will configure a static route on R1 so it knows how to reach the outside world:

<pre><code><strong>R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.12.2
</strong></code></pre>

Now we can worry about the NAT commands. Let’s configure the inside and outside interfaces:

<pre><code><strong>R2(config)#interface FastEthernet 0/0
</strong><strong>R2(config-if)#ip nat inside
</strong>
<strong>R2(config)#interface FastEthernet 1/0
</strong><strong>R2(config-if)#ip nat outside
</strong></code></pre>

Now we can try some different NAT rules.

### Port forwarding using the outside IP address

We will start with the most common scenario. When someone connects to TCP port 80 on the outside interface of R2 then it should be forwarded to R1. Here’s how to do it:

<pre><code><strong>R2(config)#ip nat inside source static tcp 192.168.12.1 80 192.168.23.2 80 extendable
</strong></code></pre>

The NAT rule above is pretty straight forward. Whenever someone tries to connect on TCP port 80 with destination IP address 192.168.23.2 then it will be forwarded to 192.168.12.1. Let’s see if it works:

<pre><code><strong>R1(config)#ip http server
</strong><strong>R1(config)#exit
</strong>
<strong>R1#debug ip http all
</strong></code></pre>

Let’s enable the HTTP server on R1 and enable a debug, we’ll be able to see when someone tries to connect. We’ll telnet from R3 to TCP port 80:

<pre><code><strong>R3#telnet 192.168.23.2 80
</strong>Trying 192.168.23.2, 80 ... Open
GET / HTTP/1.0

HTTP/1.1 401 Unauthorized
Date: Fri, 01 Mar 2002 00:09:26 GMT
Server: cisco-IOS
Accept-Ranges: none
WWW-Authenticate: Basic realm="level_15_access"

401 Unauthorized

[Connection to 192.168.23.2 closed by foreign host]
</code></pre>

R3 is able to connect and to do a HTTP GET request. On the console of R1 we will see this:

```
R1#
Fri, 01 Mar 2002 00:09:26 GMT 192.168.23.3  auth_required
        Protocol = HTTP/1.0  Method = GET
```

This proves that our port forwarding is working. R3 is able to connect and R1 sees the connection. We can also verify this by checking the NAT table on R2:

<pre><code><strong>R2#show ip nat translations
</strong>Pro Inside global      Inside local       Outside local      Outside global
tcp 192.168.23.2:80    192.168.12.1:80    192.168.23.3:39156 192.168.23.3:39156
tcp 192.168.23.2:80    192.168.12.1:80    ---                ---
</code></pre>

Above you can see that 192.168.23.2:80 will be translated to 192.168.12.1:80.

### Port forwarding using a different port

Instead of using the same port number on the outside we can also use a different port number. This is a good “security by obscurity” example, at least a non-common port won’t be scanned as often.

Let’s change our outside port from 80 to 8080:

<pre><code><strong>R2(config)#no ip nat inside source static tcp 192.168.12.1 80 192.168.23.2 80 extendable
</strong><strong>R2(config)#ip nat inside source static tcp 192.168.12.1 80 192.168.23.2 8080 extendable
</strong></code></pre>

To test this, we’ll telnet from R3 to the outside IP address of R2 on TCP port 8080:

<pre><code><strong>R3#telnet 192.168.23.2 8080
</strong>Trying 192.168.23.2, 8080 ... Open
</code></pre>

It’s able to connect. We can also verify our work on R2:

```
R2#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
tcp 192.168.23.2:8080  192.168.12.1:80    192.168.23.3:51217 192.168.23.3:51217
tcp 192.168.23.2:8080  192.168.12.1:80    ---                ---
```

Above you can see that 192.168.23.2:8080 is translated to 192.168.12.1:80.

### Port forwarding using a different IP address

In the previous two examples we used the IP address on the outside interface of R2. It’s also possible to use another IP address, for example let’s pick 192.168.23.200:

<pre><code><strong>R2(config)#no ip nat inside source static tcp 192.168.12.1 80 192.168.23.2 8080 extendable
</strong><strong>R2(config)#ip nat inside source static tcp 192.168.12.1 80 192.168.23.200 80 extendable
</strong></code></pre>

Whenever someone connects to 192.168.23.200 TCP port 80 it will be forwarded to R1:

<pre><code><strong>R3#telnet 192.168.23.200 80
</strong>Trying 192.168.23.200, 80 ... Open
</code></pre>

Here’s the NAT table on R2:

```
R2#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
tcp 192.168.23.200:80  192.168.12.1:80    192.168.23.3:59205 192.168.23.3:59205
tcp 192.168.23.200:80  192.168.12.1:80    ---                ---
```

That’s all there is to it.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
```
{% endtab %}

{% tab title="R2" %}
```
```
{% endtab %}

{% tab title="R3" %}
```
```
{% endtab %}
{% endtabs %}

I hope these examples have been useful to understand NAT port forwarding on Cisco IOS routers. If you have any questions, feel free to leave a comment.
