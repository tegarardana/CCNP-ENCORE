# EIGRP Summarization

In this lesson we’ll take a look at EIGRP summarization. The cool thing about EIGRP and summarization is that it’s easy to do and can be done on the interface level. Here’s the topology that we’ll use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-summarization-lab-topology-1.png" alt=""><figcaption></figcaption></figure>

Let’s create a basic EIGRP configuration:

<pre><code><strong>R1(config)#router eigrp 1
</strong><strong>R1(config-router)#no auto-summary 
</strong><strong>R1(config-router)#network 192.168.12.0
</strong><strong>R1(config-router)#network 172.16.0.0
</strong></code></pre>

<pre><code><strong>R2(config)#router eigrp 1
</strong><strong>R2(config-router)#no auto-summary 
</strong><strong>R2(config-router)#network 192.168.12.0
</strong></code></pre>

Just a basic EIGRP configuration. This is what the routing table of R2 looks like:

<pre><code><strong>R2#show ip route eigrp 
</strong><strong>     172.16.0.0/24 is subnetted, 2 subnets
</strong>D       172.16.0.0 [90/30720] via 192.168.12.1, 00:02:28, FastEthernet0/0
D       172.16.1.0 [90/30720] via 192.168.12.1, 00:02:28, FastEthernet0/0
</code></pre>

Two entries, as expected. Now let’s create that summary:

<pre><code><strong>R1(config)#interface fastEthernet 2/0
</strong><strong>R1(config-if)#ip summary-address eigrp 1 172.16.0.0 255.255.254.0
</strong></code></pre>

Use the `ip summary-address eigrp` command to specify the summary. Now the routing table will look like this:

<pre><code><strong>R2#show ip route eigrp 
</strong><strong>     172.16.0.0/23 is subnetted, 1 subnets
</strong>D       172.16.0.0 [90/30720] via 192.168.12.1, 00:00:53, FastEthernet0/0
</code></pre>

The two networks disappear, and a single entry will appear on R2. The metric is 30720. How did it get this value? When you advertise a summary, the router advertising the summary will use the lowest metric of all networks that fall within the range of the summary. That’s the one it will use for the summary. It is possible to manually set the metric for each summary. Here’s how:

<pre><code><strong>R1(config)#router eigrp 1
</strong><strong>R1(config-router)#summary-metric ?
</strong></code></pre>

The `summary-metric` command requires you to set the bandwidth, delay, reliability, load, and MTU. Let’s try something:

<pre><code><strong>R1(config-router)#summary-metric 172.16.0.0 255.255.254.0 100000 10 255 0 1500
</strong></code></pre>

R2 will now see the new metric:

<pre><code><strong>R2#show ip route eigrp 
</strong><strong>     172.16.0.0/23 is subnetted, 1 subnets
</strong>D       172.16.0.0 [90/2562816] via 192.168.12.1, 00:00:53, FastEthernet0/0
</code></pre>

Advertising a summary also changes something in the routing table of R1:

<pre><code><strong>R1#show ip route eigrp 
</strong>     172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
D       172.16.0.0/23 is a summary, 00:01:38, Null0
</code></pre>

EIGRP will create an entry in the routing table for the summary pointing to the null0 interface. It does this to protect itself against routing loops. When you use summaries, other routers may send traffic to you for networks you don’t know where they are. When this happens, the traffic will be forwarded to the null0 interface and dropped.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface FastEthernet2/0
 ip address 192.168.12.1 255.255.255.0
 ip summary-address eigrp 1 172.16.0.0 255.255.254.0
!
interface FastEthernet0/0
 ip address 172.16.0.1 255.255.255.0
!
interface FastEthernet1/0
 ip address 172.16.1.1 255.255.255.0
!
router eigrp 1
 network 172.16.0.0
 network 192.168.12.0
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface FastEthernet0/0
 ip address 192.168.12.2 255.255.255.0
!
router eigrp 1
 network 192.168.12.0
!
end
```
{% endtab %}
{% endtabs %}
