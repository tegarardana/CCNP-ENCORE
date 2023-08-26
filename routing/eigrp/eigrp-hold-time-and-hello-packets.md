# EIGRP Hold Time and Hello Packets

Most people have learned that the EIGRP hold timer is renewed when it receives a hello packet from a neighbor. This is correct however the hello packet is not the only packet that renewes it…all EIGRP packets do. In order to demonstrate this, we’ll take a look at the following two routers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/09/R1-R2-EIGRP-AS-12.png" alt=""><figcaption></figcaption></figure>

First, I will configure EIGRP on both routers, nothing special I want to make sure we have a neighbor adjacency:

<pre><code><strong>R1(config)#router eigrp 12
</strong><strong>R1(config-router)#network 192.168.12.0
</strong></code></pre>

<pre><code><strong>R2(config)#router eigrp 12
</strong><strong>R2(config-router)#network 192.168.12.0
</strong></code></pre>

Now I will increase the hold time so it doesn’t drop the neighbor adjacency so quickly. I’ll set it to 1 hour:

<pre><code><strong>R2(config-if)#ip hold-time eigrp 12 3600
</strong></code></pre>

When we take a look at R1, you’ll see that it uses 3600 seconds as the hold time for R2:

<pre><code><strong>R1#show ip eigrp neighbors 
</strong>IP-EIGRP neighbors for process 12
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
0   192.168.12.2            Fa0/0            3597 00:00:51    3   200  0  21
</code></pre>

We have 3597 seconds and counting…now I will set the hello timer to a low value so that we can find out if other EIGRP packets will renew the hold time. I’ll set it to 5 minutes:

<pre><code><strong>R2(config)#interface fastEthernet 0/0
</strong><strong>R2(config-if)#ip hello-interval eigrp 12 300
</strong></code></pre>

R1 will now only receive a hello packet from R2 every 5 minutes. To show that different EIGRP packets will renew the hold time, I will enable a debug on R1 so that we can see what kind of packets it receives from R2:

<pre><code><strong>R1#debug eigrp packets 
</strong>EIGRP Packets debugging is on
    (UPDATE, REQUEST, QUERY, REPLY, HELLO, IPXSAP, PROBE, ACK, STUB, SIAQUERY, SIAREPLY)
</code></pre>

Now I will create a new loopback interface on R2 and advertise it in EIGRP. This will cause some traffic between R1 and R2. Before I do this, let’s take a quick look at the current state of the hold time again:

Look again at the table:

<pre><code><strong>R1#show ip eigrp neighbors 
</strong>IP-EIGRP neighbors for process 12
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
0   192.168.12.2            Fa0/0            3504 00:04:23    3   200  0  21
</code></pre>

Right now, we are down to 3504 seconds…let’s advertise that loopback interface in EIGRP:

<pre><code><strong>R2(config)#interface loopback 0
</strong><strong>R2(config-if)#ip address 2.2.2.2 255.255.255.0
</strong></code></pre>

<pre><code><strong>R2(config)#router eigrp 12
</strong><strong>R2(config-router)#network 2.0.0.0
</strong></code></pre>

We’ll see on the console of R1 that it receives an UPDATE message, some ACKS, and that’s it…

```
R1#
EIGRP: Received UPDATE on FastEthernet0/0 nbr 192.168.12.2
  AS 12, Flags 0x0, Seq 22/0 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 0/0
EIGRP: Enqueueing ACK on FastEthernet0/0 nbr 192.168.12.2
  Ack seq 22 iidbQ un/rely 0/0 peerQ un/rely 1/0
EIGRP: Sending ACK on FastEthernet0/0 nbr 192.168.12.2
  AS 12, Flags 0x0, Seq 0/22 idbQ 0/0 iidbQ un/rely 0/0 peerQ un/rely 1/0
EIGRP: Enqueueing UPDATE on FastEthernet0/0 iidbQ un/rely 0/1 serno 3-3
EIGRP: Enqueueing UPDATE on FastEthernet0/0 nbr 192.168.12.2 iidbQ un/rely 0/0 peerQ un/rely 0/0 serno 3-3
```

What about the hold time? Did it reset? Let’s find out:

<pre><code><strong>R1#show ip eigrp neighbors 
</strong>IP-EIGRP neighbors for process 12
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
0   192.168.12.2            Fa0/0            3588 00:05:01    3   300  0  22
</code></pre>

There we go. The hold time was reset even though we didn’t receive any hello packets from R2. The other messages did their work just fine…

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet0/0
 bandwidth 500
 ip address 192.168.12.1 255.255.255.0
 delay 50
!         
router eigrp 12
 network 192.168.12.0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.0
!
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
 ip hello-interval eigrp 12 300
 ip hold-time eigrp 12 3600
!
router eigrp 12
 network 2.0.0.0
 network 192.168.12.0
!
end
```
{% endtab %}
{% endtabs %}

I hope this has been a useful lesson for you. If you have any questions feel free to ask!
