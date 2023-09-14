# NTPv4

NTPv4 is an extension of [NTPv3](https://networklessons.com/cisco/ccnp-encor-350-401/cisco-network-time-protocol-ntp) that supports IPv4 and IPv6. It is backward compatible with NTPv3, offers some new features, and time synchronization is faster and more precise.

Security has improved, NTPv4 supports public key cryptography and standard X509 certificates.

When using NTP for IPv4, broadcast is a popular option as it allows you to send NTP packets in the broadcast domain to everyone. We can’t do this with IPv6, but NTPv4 does support site-local multicast.

DNS support is also improved. With NTPv3, if you configure a hostname to sync with, your device does a lookup for the hostname and stores the IP address in the configuration, the hostname is then lost. With NTPv4, the hostname is stored in the configuration.

In this lesson, I’ll show you how to configure NTPv4 with a unicast and multicast client.

## Configuration

This is the topology we’ll use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/07/ntpv4-lab-topology.png" alt=""><figcaption></figcaption></figure>

Configuration-wise, NTPv4 is pretty much the same thing.

To help speed things up, let’s set the same time and date on all routers before we configure NTP:

<pre><code>R1, R2 &#x26; R3
<strong>#clock set 10:37:00 2 July 2018
</strong></code></pre>

I will configure R1 as an NTP master so that I don’t need an external server:

<pre><code><strong>R1(config)#ntp master 1
</strong></code></pre>

### Clients

Let’s configure our clients. R2 will be an NTP unicast client and for R3 we will use multicast.

#### **Unicast**

We can configure the IPv6 address of R1 but instead, we’ll use a hostname to test if R2 stores the hostname in its configuration. I’ll create a manual host record for this:

<pre><code><strong>R2(config)#ipv6 host R1 2001:DB8:0:12::1
</strong></code></pre>

Now we configure R1 as the NTP server. The `version 4` parameter sets the correct version:

<pre><code><strong>R2(config)#ntp server R1 version 4
</strong></code></pre>

#### **Multicast**

To make multicast work, we need to configure R1 to send NTP multicast packet and R3 to receive them.

This is the multicast address we will use:

FF05::101

* FF05 is the multicast address for the [site-local scope](https://www.iana.org/assignments/ipv6-multicast-addresses/ipv6-multicast-addresses.xhtml#site-local).
* ::101 is the address that IANA has assigned to NTP for IPv6.

Let’s configure R1 to send NTP multicast packets with this address:

<pre><code><strong>R1(config)#interface FastEthernet 0/1 
</strong><strong>R1(config-if)#ntp multicast FF05::101 version 4
</strong></code></pre>

and R3 to receive multicast NTP packets:

<pre><code><strong>R3(config)#interface FastEthernet 0/0
</strong><strong>R3(config-if)#ntp multicast client FF05::101
</strong></code></pre>

## Verification

Let’s verify our work. R1 is synchronized:

<pre><code><strong>R1#show ntp status | include synchronized
</strong>Clock is synchronized, stratum 1, reference is .LOCL.
</code></pre>

R1 has a stratum of 1 because that’s how I configured it as a master. Let’s check R2:

<pre><code><strong>R2#show ntp status | include synchronized
</strong>Clock is synchronized, stratum 2, reference is 48.49.82.9 
</code></pre>

R2 is synchronized. So is R3:

<pre><code><strong>R3#show ntp status | include synchronized
</strong>Clock is synchronized, stratum 2, reference is 191.117.190.88
</code></pre>

All our clocks are synchronized. Let’s take a closer look at the NTP associations:

<pre><code><strong>R1#show ntp associations
</strong>
  address         ref clock       st   when   poll reach  delay  offset   disp
*~127.127.1.1     .LOCL.           0      6     16   377  0.000   0.000  0.245
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
</code></pre>

Above, we see that R1 uses its own address to synchronize with. Here’s R2:

<pre><code><strong>R2#show ntp associations 
</strong>
  address         ref clock       st   when   poll reach  delay  offset   disp
*~2001:DB8:0:12::1
                  .LOCL.           1     21     64     1  2.573   5.507 7937.5
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
</code></pre>

R2 uses the IPV6 address of R1 as its NTP server. If you look in the running configuration, you will see the hostname:

<pre><code><strong>R2#show running-config | include ntp
</strong>ntp server R1
</code></pre>

Here is R3:

<pre><code><strong>R3#show ntp associations
</strong>
  address         ref clock       st   when   poll reach  delay  offset   disp
* 2001:DB8:0:13::1
                  .LOCL.           1     22     64     1  0.000  -5.888 7937.5
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
</code></pre>

R3 doesn’t show the `~` symbol since it receives the NTP packets through multicast. It does show the IPv6 address of R1 though.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
ipv6 cef
!
interface FastEthernet0/0
 ipv6 address 2001:DB8:0:12::1/64
!
interface FastEthernet0/1
 ipv6 address 2001:DB8:0:13::1/64
 ntp multicast FF05::101
!
ntp master 1
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
ip host R1 2001:DB8:0:12::1
ipv6 cef
!
interface FastEthernet0/0
 ipv6 address 2001:DB8:0:12::2/64
!
ntp server R1
!
end
```
{% endtab %}

{% tab title="R3" %}
```
hostname R3
!
ipv6 cef
!
interface FastEthernet0/0
 ipv6 address 2001:DB8:0:13::3/64
 ntp multicast client FF05::101
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

You have now learned how to configure NTPv4 using IPv6 unicast and multicast.

If you want to learn more details about NTPv4, take a look at [RFC5905](https://tools.ietf.org/html/rfc5905).

I hope you enjoyed this lesson. If you have any questions feel free to leave a comment!
