# BGP Attribute Origin Cod

The BGP Origin Code is one of the attributes that is used for path selection. There are three origin codes that the BGP table can show:

* IGP (shows up as i)
* EGP (shows up as e)
* Incomplete (shows up as ?)

You will see IGP when you use the `network` command for BGP. It means you advertised the network yourself in BGP. EGP is historical, and you won’t see it in the BGP table anymore. EGP is an old routing protocol. We don’t use it anymore. Incomplete means you have redistributed something into BGP.

## Configuration

Here’s a demonstration:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/bgp-as-path-prepend-lab.png" alt=""><figcaption></figcaption></figure>

Above, you can see the topology that I will use. R1 and R3 are in AS1 and connected to R2 in AS2. Both routers have a loopback0 interface with network 1.1.1.0/24 configured on it. Let’s configure BGP:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#neighbor 192.168.12.2 remote-as 2
</strong></code></pre>

<pre><code><strong>R3(config)#router bgp 1
</strong><strong>R3(config-router)#neighbor 192.168.23.2 remote-as 2
</strong></code></pre>

<pre><code><strong>R2(config)#router bgp 2
</strong><strong>R2(config-router)#neighbor 192.168.12.1 remote-as 1
</strong><strong>R2(config-router)#neighbor 192.168.23.3 remote-as 1
</strong></code></pre>

The next step is to get network 1.1.1.0/24 in the BGP table:

<pre><code><strong>R1(config)#router bgp 1
</strong><strong>R1(config-router)#network 1.1.1.0 mask 255.255.255.0
</strong></code></pre>

<pre><code><strong>R3(config)#router bgp 1
</strong><strong>R3(config-router)#redistribute connected
</strong></code></pre>

On R1, I’ll advertise network 1.1.1.0/24 in BGP with the network command. On R3, we’ll redistribute it. Let’s see what R2 thinks of this…

<pre><code><strong>R2#show ip bgp 
</strong>BGP table version is 4, local router ID is 192.168.23.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  1.1.1.0/24       192.168.23.3             0             0 1 ?
*>                  192.168.12.1             0             0 1 i
</code></pre>

In the output above, you can see that R2 learned both networks through BGP. There’s one small difference, however. The first entry shows a `?` symbol and the second entry shows an ‘i’.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface Loopback 0
 ip address 1.1.1.1 255.255.255.0
!
interface fastEthernet2/0
 ip address 192.168.12.1 255.255.255.0
!
router bgp 1
 neighbor 192.168.12.2 remote-as 2
 network 1.1.1.0 mask 255.255.255.0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface fastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
interface fastEthernet1/0
 ip address 192.168.23.2 255.255.255.0
!
router bgp 2
 neighbor 192.168.12.1 remote-as 1
 neighbor 192.168.23.3 remote-as 1
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
interface Loopback 0
 ip address 1.1.1.1 255.255.255.0
!
interface fastEthernet2/0
 ip address 192.168.23.3 255.255.255.0
!
router bgp 1
 neighbor 192.168.23.2 remote-as 2
 redistribute connected
!
end
```
{% endtab %}
{% endtabs %}

That’s all there is to it. If you have any more questions, leave a comment!
