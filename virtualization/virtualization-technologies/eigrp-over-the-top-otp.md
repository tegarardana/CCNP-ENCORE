# EIGRP Over The Top (OTP)

EIGRP OTP (Over the Top) allows you to run EIGRP between routers that are **not directly connected**. It is a great feature to use when you want to run EIGRP between routers that are connected to a service provider network and you don’t want the hassle of other solutions like [MPLS VPN](https://networklessons.com/mpls/mpls-layer-3-vpn-pe-ce-eigrp/) and you don’t want the service provider’s involvement with your routing. With EIGRP OTP, you can configure everything on your own routers as long as you are able to reach the remote router(s).

EIGRP OTP uses an overlay VPN and thus uses tunneling, it is a bit similar to other solutions like DMVPN / multipoint GRE but it uses [**LISP (Locator/Identifier Separation Protocol)**](https://networklessons.com/cisco/ccnp-encor-350-401/cisco-locator-id-separation-protocol-lisp) instead.

LISP is a technology where each end device has a separate “identity” and “location”, unlike IP where the IP address is both the identity and location of the end device. LISP is useful in scenarios where an end device should be able to retain its original IP address, even if it moves to another subnet. It is useful for VM motion where virtual machines are moved from one hypervisor to another but also for mobile devices that end up in another subnet after roaming.

There is a control plane and data plane for LISP. Learning all the details about the LISP control plane is quite a story. On the data plane, LISP uses UDP to encapsulate data and tunnel traffic. EIGRP OTP **only uses LISP for the data plane**, **EIGRP is used for the control plane.** This is convenient as you don’t really have to understand LISP to be able to understand EIGRP OTP.

In this lesson, I’ll show you how to configure EIGRP OTP.

## Configuration

To demonstrate EIGRP OTP, I use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/09/eigrp-otp-demo-topology.png" alt=""><figcaption></figcaption></figure>

Above we have four routers:

* R1, R2 and R3 are customer routers that want to use EIGRP to exchange routing information. These routers each have a loopback interface with an IP address that we will advertise in EIGRP.
* R4 is the service provider network. In a production network, this could be a large network but for simplicity reasons, I’m only using one router here.

The only requirement to run EIGRP OTP is that the customer routers have to be able to reach each other. I will use a couple of static routes so that R1, R2 and R3 are able to reach the IP addresses on their GigabitEthernet 0/1 interfaces:

<pre><code><strong>R1(config)#ip route 192.168.24.2 255.255.255.255 192.168.14.4
</strong><strong>R1(config)#ip route 192.168.34.3 255.255.255.255 192.168.14.4
</strong></code></pre>

<pre><code><strong>R2(config)#ip route 192.168.14.1 255.255.255.255 192.168.24.4
</strong><strong>R2(config)#ip route 192.168.34.3 255.255.255.255 192.168.24.4
</strong></code></pre>

<pre><code><strong>R3(config)#ip route 192.168.14.1 255.255.255.255 192.168.34.4
</strong><strong>R3(config)#ip route 192.168.24.2 255.255.255.255 192.168.34.4
</strong></code></pre>

Now we can configure EIGRP.

### Route Reflector

EIGRP OTP is only supported in [EIGRP named mode](https://networklessons.com/eigrp/eigrp-named-mode-configuration), you can’t configure it in classic mode. Neighbors have to be configured statically and we have two options here:

* We can configure a full-mesh of static EIGRP neighbors.
* We can configure one router as a route reflector which works similar to the [BGP route reflector](https://networklessons.com/eigrp/bgp-route-reflector).

With three routers, a full mesh is no problem but if you have a lot of customer routers, a route reflector is an interesting option. I will configure R1 as the route reflector and R2/R3 will be its clients:

<pre><code><strong>R1(config)#router eigrp OTP
</strong><strong>R1(config-router)#address-family ipv4 autonomous-system 123
</strong><strong>R1(config-router-af)#remote-neighbors source GigabitEthernet 0/1 unicast-listen lisp-encap 123 
</strong></code></pre>

The **remote-neighbors** command is all we need for EIGRP OTP. It tells R1 to use the GigabitEthernet 0/1 interface and listen for unicast packets. LISP uses different IDs, I’m going to use ID 123 on all routers. Optionally, this command allows you to set a maximum number of neighbors and/or use an access-list to permit only certain IP addresses to accept as EIGRP neighbors.

Let’s advertise the loopback interface and GigabitEthernet 0/1 on R1:

<pre><code><strong>R1(config-router-af)#network 1.1.1.1 0.0.0.0
</strong><strong>R1(config-router-af)#network 192.168.14.0 0.0.0.255
</strong></code></pre>

Last but not least, there are two more things we have to configure:

* **Disable next-hop-self**: If you don’t do this, R1 will set its own IP address as the next hop for everything it advertises to R2 and R3. If you don’t disable this, then whenever R2 or R3 want to reach each other’s loopback interface, they will send their traffic to R1. It works but it’s sub-optimal routing. If you disable this then R1 will leave the next hop IP address alone and R2/R3 are able to reach each other directly.
* **Disable split horizon**: R1 is using a single interface so if you don’t disable split horizon, R2 and R3 are unable to learn each others loopback interfaces.

Let’s disable next-hop-self and split horizon:

<pre><code><strong>R1(config-router-af)#af-interface GigabitEthernet 0/1
</strong><strong>R1(config-router-af-interface)#no next-hop-self 
</strong><strong>R1(config-router-af-interface)#no split-horizon 
</strong></code></pre>

That’s all we have to do on the route reflector.

### Spoke routers

Let’s configure R2 and R3. The only thing we have to do is configure R1 as a static remote neighbor and set the LISP encapsulation ID:

<pre><code><strong>R2(config)#router eigrp OTP
</strong><strong>R2(config-router)#address-family ipv4 autonomous-system 123
</strong><strong>R2(config-router-af)#neighbor 192.168.14.1 GigabitEthernet 0/1 remote lisp-encap 123
</strong><strong>R2(config-router-af)#network 2.2.2.2 0.0.0.0
</strong><strong>R2(config-router-af)#network 192.168.24.0 0.0.0.255
</strong></code></pre>

<pre><code><strong>R3(config)#router eigrp OTP
</strong><strong>R3(config-router)#address-family ipv4 autonomous-system 123
</strong><strong>R3(config-router-af)#neighbor 192.168.14.1 GigabitEthernet 0/1 remote lisp-encap 123
</strong><strong>R3(config-router-af)#network 3.3.3.3 0.0.0.0
</strong><strong>R3(config-router-af)#network 192.168.34.0 0.0.0.255
</strong></code></pre>

That’s all there is to it.

## Verification

Let’s verify our work. The first thing you notice once you configure OTP is that the routers create a new LISP interface:

```
R1, R2 & R3#
%LINEPROTO-5-UPDOWN: Line protocol on Interface LISP123, changed state to up
```

We can see it here:

<pre><code><strong>R1#show ip interface brief
</strong>Interface                  IP-Address      OK? Method Status                Protocol  
GigabitEthernet0/1         192.168.14.1    YES NVRAM  up                    up      
LISP123                    1.1.1.1         YES unset  up                    up      
Loopback0                  1.1.1.1         YES NVRAM  up                    up
</code></pre>

Let’s see if we have EIGRP neighbors:

<pre><code><strong>R1#show ip eigrp neighbors 
</strong>EIGRP-IPv4 VR(OTP) Address-Family Neighbors for AS(123)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   192.168.24.2            Gi0/1                    14 00:02:42  476  2856  0  3
0   192.168.34.3            Gi0/1                    14 00:02:49  296  1776  0  4
</code></pre>

<pre><code><strong>R2#show ip eigrp neighbors 
</strong>EIGRP-IPv4 VR(OTP) Address-Family Neighbors for AS(123)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
0   192.168.14.1            Gi0/1                    13 00:02:54  211  1266  0  6
</code></pre>

<pre><code><strong>R3#show ip eigrp neighbors 
</strong>EIGRP-IPv4 VR(OTP) Address-Family Neighbors for AS(123)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
0   192.168.14.1            Gi0/1                    14 00:03:08  214  1284  0  6
</code></pre>

R1 has established neighbor adjacencies with both R2 and R3. Let’s check the routing tables:

<pre><code><strong>R1#show ip route eigrp 
</strong>
      2.0.0.0/32 is subnetted, 1 subnets
D        2.2.2.2 [90/93994331] via 192.168.24.2, 00:03:33, LISP123
      3.0.0.0/32 is subnetted, 1 subnets
D        3.3.3.3 [90/93994331] via 192.168.34.3, 00:03:38, LISP123
</code></pre>

<pre><code><strong>R2#show ip route eigrp 
</strong>
      1.0.0.0/32 is subnetted, 1 subnets
D        1.1.1.1 [90/93994331] via 192.168.14.1, 00:03:50, LISP123
      3.0.0.0/32 is subnetted, 1 subnets
D        3.3.3.3 [90/93994331] via 192.168.34.3, 00:03:50, LISP123
</code></pre>

<pre><code><strong>R3#show ip route eigrp 
</strong>
      1.0.0.0/32 is subnetted, 1 subnets
D        1.1.1.1 [90/93994331] via 192.168.14.1, 00:00:06, LISP123
      2.0.0.0/32 is subnetted, 1 subnets
D        2.2.2.2 [90/93994331] via 192.168.24.2, 00:00:06, LISP123
</code></pre>

Above you can see that all routers have learned about the loopback interfaces. Note that we are using the LISP123 interface. Because we disabled next-hop-self, R2 and R3 have the correct next hop IP addresses which allow them to reach each other directly.

If you look in the CEF table, you can see how the routers figure out how to reach each other:

<pre><code><strong>R1#show ip cef 2.2.2.2 internal 
</strong>2.2.2.2/32, epoch 0, RIB[I], refcnt 5, per-destination sharing
  sources: RIB 
  feature space:
    IPRM: 0x00028000
  ifnums:
    LISP123(8): 192.168.24.2
  path list 0EBF8D04, 3 locks, per-destination, flags 0x49 [shble, rif, hwcn]
    path 0EBF9254, share 1/1, type attached nexthop, for IPv4
      nexthop 192.168.24.2 LISP123, IP midchain out of LISP123, addr 192.168.24.2 0E0ACE08
  output chain:
    IP midchain out of LISP123, addr 192.168.24.2 0E0ACE08
    IP adj out of GigabitEthernet0/1, addr 192.168.14.4 0D800840
</code></pre>

Above we see that R1 uses the LISP123 interface to reach 2.2.2.2 which uses 192.168.24.2 (R2) as the next hop. When R1 tries to send something to 2.2.2.2, it will use LISP encapsulation which I will show you in a bit.

First, let’s see if we have connectivity:

<pre><code><strong>R2#ping 1.1.1.1 source 2.2.2.2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
Packet sent with a source address of 2.2.2.2 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/8/10 ms
</code></pre>

R2 is able to reach R1, let’s see if it can reach R3:

<pre><code><strong>R2#ping 3.3.3.3 source 2.2.2.2
</strong>Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
Packet sent with a source address of 2.2.2.2 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/11/15 ms
</code></pre>

That’s also no problem. If you enable a packet capture, you can see that EIGRP control plane traffic like hello packets or updates are sent as regular IP packets. This is possible since we use EIGRP unicast traffic:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/09/eigrp-otp-hello-packet.png" alt=""><figcaption></figcaption></figure>

Traffic on the data plane, like our pings from one loopback interface to another are encapsulated with LISP. Here’s a capture:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/09/eigrp-otp-lisp-encapsulated-icmp-request.png" alt=""><figcaption></figcaption></figure>

Above you can see that the packet is sent between 192.168.14.1 (R1) and 192.168.24.2 (R2). The actual ICMP traffic however is encapsulated with LISP in UDP using destination port 4342.

[EIGRP OTP Data Plane](https://www.cloudshark.org/captures/1929edca36b3)

That’s all there is to it.

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
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface LISP123
!
interface GigabitEthernet0/1
 ip address 192.168.14.1 255.255.255.0
!
router eigrp OTP
 !
 address-family ipv4 unicast autonomous-system 123
  !
  af-interface GigabitEthernet0/1
   no next-hop-self
   no split-horizon
  exit-af-interface
  !
  topology base
  exit-af-topology
  remote-neighbors source GigabitEthernet0/1 unicast-listen lisp-encap 123
  network 1.1.1.1 0.0.0.0
  network 192.168.14.0
 exit-address-family
!
ip route 192.168.24.2 255.255.255.255 192.168.14.4
ip route 192.168.34.3 255.255.255.255 192.168.14.4
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
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
interface LISP123
!
interface GigabitEthernet0/1
 ip address 192.168.24.2 255.255.255.0
!
router eigrp OTP
 !
 address-family ipv4 unicast autonomous-system 123
  !
  topology base
  exit-af-topology
  neighbor 192.168.14.1 GigabitEthernet0/1 remote 100 lisp-encap 123 
  network 2.2.2.2 0.0.0.0
  network 192.168.24.0
 exit-address-family
!
ip route 192.168.14.1 255.255.255.255 192.168.24.4
ip route 192.168.34.3 255.255.255.255 192.168.24.4
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
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
!
interface LISP123
!
interface GigabitEthernet0/1
 ip address 192.168.34.3 255.255.255.0
!
router eigrp OTP
 !
 address-family ipv4 unicast autonomous-system 123
  !
  topology base
  exit-af-topology
  neighbor 192.168.14.1 GigabitEthernet0/1 remote 100 lisp-encap 123
  network 3.3.3.3 0.0.0.0
  network 192.168.34.0
 exit-address-family
!
ip route 192.168.14.1 255.255.255.255 192.168.34.4
ip route 192.168.24.2 255.255.255.255 192.168.34.4
!
end
```
{% endtab %}

{% tab title="R4" %}
```
hostname R4
!
ip cef
!
interface GigabitEthernet0/1
 ip address 192.168.14.4 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 192.168.24.4 255.255.255.0
!
interface GigabitEthernet0/3
 ip address 192.168.34.4 255.255.255.0
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

You have now learned:

* How EIGRP OTP allows you to establish neighbor adjacencies between remote routers by using LISP encapsulation.
* That EIGRP OTP is configured entirely from within EIGRP, which is quite convenient compared to other solutions like MPLS VPN PE-CE or DMVPN.
* That you can use a route reflector for large EIGRP OTP topologies.
* How to verify that EIGRP is configured correctly and working.
* How EIGRP OTP packets on the data plane are encapsulated in UDP and use destination port 4342.
