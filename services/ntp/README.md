# NTP

## Cisco Network Time Protocol (NTP)

NTP (Network Time Protocol) is used to allow network devices to synchronize their clocks with a central source clock. For network devices like routers, switches, or firewalls this is very important because we want to make sure that logging information and timestamps have the accurate time and date. If you ever have network issues or get hacked, you want to know exactly what and _when_ it happened.

Normally a router or switch will run in NTP client mode, which means that it will adjust its clock based on the time of an NTP server. Basically the NTP protocol describes the algorithm that the NTP clients use to synchronize their clocks with the NTP server and the packets that are used between them.

A good example of a NTP server is [ntp.pool.org](http://www.pool.ntp.org/). This is a cluster of NTP servers that many servers and network devices use to synchronize their clocks.

NTP uses a concept called “stratum” that defines how many NTP hops away a device is from an authoritative time source. For example, a device with stratum 1 is a very accurate device and might have an atomic clock attached to it. Another NTP server using this stratum 1 server to sync its own time would be a stratum two device because it’s one NTP hop further away from the source. When you configure multiple NTP servers, the client will prefer the NTP server with the lowest stratum value.

Cisco routers and switches can use three different NTP modes:

* NTP client mode.
* NTP server mode.
* NTP symmetric active mode.

The symmetric active mode is used between NTP devices to synchronize with each other, it’s used as a backup mechanism when they are unable to reach the (external) NTP server.

In the remainder of this lesson, I will demonstrate how to configure NTP on a Cisco router and switches.

### Configuration

This is the topology I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/07/cisco-ntp-example-topology.png" alt=""><figcaption></figcaption></figure>

The router on the top is called “CoreRouter” and it’s the edge of my network. It is connected to the Internet and will use one of the NTP servers from pool.ntp.org to synchronize its clock. The network also has two internal switches that require synchronized clocks. Both switches will become NTP clients of the CoreRouter, thus making the CoreRouter an NTP server.

#### Router configuration

First, we will configure the CoreRouter on top. I will use pool.ntp.org as the external NTP server for this example. We need to make sure that the router can resolve hostnames:

<pre><code><strong>CoreRouter(config)#ip name-server 8.8.8.8
</strong></code></pre>

I will use Google DNS for this. Our next step is to configure the NTP server:

<pre><code><strong>CoreRouter(config)#ntp server pool.ntp.org
</strong></code></pre>

That was easy enough, just one command and we will synchronize our clock with the public server. We can verify our work like this:

<pre><code><strong>CoreRouter#show ntp associations 
</strong>
  address         ref clock       st   when   poll reach  delay  offset   disp
 ~146.185.130.223 .INIT.          16      -     64     0  0.000   0.000 16000.
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
</code></pre>

Above we see the **`show ntp associations`** command that tells us if our clock is synchronized or not. The \~ in front of the IP address tells us that we configured this server but we are _not synchronized yet_. You can see this because there is no \* in front of the IP address, and the “st” field (stratum) is currently 16.

There is one more command that gives us more information about the NTP configuration:

<pre><code><strong>CoreRouter#show ntp status
</strong>Clock is unsynchronized, stratum 16, no reference clock
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**24
reference time is 00000000.00000000 (00:00:00.000 UTC Mon Jan 1 1900)
clock offset is 0.0000 msec, root delay is 0.00 msec
root dispersion is 0.16 msec, peer dispersion is 0.00 msec
loopfilter state is 'FSET' (Drift set from file), drift is 0.000000000 s/s
system poll interval is 64, never updated.
</code></pre>

The router tells us that we are unsynchronized and that there is no reference clock…we will wait a couple of minutes and take a look at these commands again:

<pre><code><strong>CoreRouter#show ntp associations 
</strong>
  address         ref clock       st   when   poll reach  delay  offset   disp
*~146.185.130.223 193.79.237.14    2     26     64     1 10.857  -5.595 7937.5
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
</code></pre>

A few minutes later the output has changed. The \* in front of the IP address tells us that we have synchronized and the stratum is 2…that means that this NTP server is pretty close to a reliable time source. The “poll” field tells us we will try synchronizing the time every 64 seconds. Let’s check the other command that we just saw:

<pre><code><strong>CoreRouter#show ntp status       
</strong>Clock is synchronized, stratum 3, reference is 146.185.130.22
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**24
reference time is D76513B4.66A4CDA6 (12:40:20.400 UTC Mon Jul 7 2014)
clock offset is -5.5952 msec, root delay is 13.58 msec
root dispersion is 7966.62 msec, peer dispersion is 7937.50 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is -0.000000018 s/s
system poll interval is 64, last update was 43 sec ago.
</code></pre>

Our clock has been synchronized, and our own stratum is 3, that makes sense since the public stratum server has a stratum of 2 and we are one “hop” away from it.

{% hint style="info" %}
NTP synchronization can be very slow, so you have to be patient when your clocks are not synchronized. One way to speed it up a bit is to adjust your clock manually so it is closer to the current time.
{% endhint %}

Cisco routers have two different clocks; they have a software clock and a hardware clock, and they operate separately from each other. Here’s how to see both clocks:

<pre><code><strong>CoreRouter#show clock
</strong>12:41:25.197 UTC Mon Jul 7 2014
</code></pre>

<pre><code><strong>CoreRouter#show calendar
</strong>12:43:24 UTC Mon Jul 7 2014
</code></pre>

The **`show clock command`** shows me the software clock, while the **`show calendar command`** gives me the hardware clock. The two clocks are not in sync, so this is something we should fix. You can do it like this:

<pre><code><strong>CoreRouter#(config)ntp update-calendar
</strong></code></pre>

The **`ntp update-calendar`** command will update the hardware clock with the time of the software clock. Here’s the result:

<pre><code><strong>CoreRouter#show clock
</strong>12:42:31.853 UTC Mon Jul 7 2014
</code></pre>

<pre><code><strong>CoreRouter#show calendar
</strong>12:42:30 UTC Mon Jul 7 2014
</code></pre>

That’s all I wanted to configure on the CoreRouter for now. We still have to configure two switches to synchronize their clocks.

#### Switch Configuration

The two switches will be configured to use the CoreRouter as the NTP server, and I will also configure them to synchronize their clocks with each other. Let’s configure them to use the CoreRouter first:

<pre><code><strong>SW1(config)#ntp server 192.168.123.3
</strong></code></pre>

Once again, it might take a few minutes to synchronize, but this is what you will see:

<pre><code><strong>SW1#show ntp associations 
</strong>
      address         ref clock     st  when  poll reach  delay  offset    disp
*~192.168.123.3    146.185.130.223   3    21    64    1     2.5    1.02  15875.
 * master (synced), # master (unsynced), + selected, - candidate, ~ configured
</code></pre>

<pre><code><strong>SW1#show ntp status       
</strong>Clock is synchronized, stratum 4, reference is 192.168.123.3
nominal freq is 119.2092 Hz, actual freq is 119.2089 Hz, precision is 2**18
reference time is D765271D.D6021302 (14:03:09.835 UTC Mon Jul 7 2014)
clock offset is 1.0229 msec, root delay is 14.31 msec
root dispersion is 16036.00 msec, peer dispersion is 15875.02 msec
</code></pre>

The clock of SW1 has been synchronized, and its stratum is 4. This makes sense since it’s one “hop” further away from its NTP server (CoreRouter). Let’s do the same for SW2:

<pre><code><strong>SW2(config)#ntp server 192.168.123.3
</strong></code></pre>

Let’s be patient for a few more minutes, and this is what we’ll get:

<pre><code><strong>SW2#show ntp associations
</strong>
      address         ref clock     st  when  poll reach  delay  offset    disp
*~192.168.123.3    146.185.130.223   3    17    64   37     3.4    1.89   875.8
 * master (synced), # master (unsynced), + selected, - candidate, ~ configured
</code></pre>

<pre><code><strong>SW2#show ntp status 
</strong><strong>Clock is synchronized, stratum 4, reference is 192.168.123.3
</strong>nominal freq is 119.2092 Hz, actual freq is 119.2084 Hz, precision is 2**18
reference time is D765274D.D51A0546 (14:03:57.832 UTC Mon Jul 7 2014)
clock offset is 1.8875 msec, root delay is 15.18 msec
root dispersion is 1038.39 msec, peer dispersion is 875.84 msec
</code></pre>

SW1 and SW2 are now using CoreRouter to synchronize their clocks. Let’s also configure them to use each other for synchronization. This is the symmetric active mode I mentioned before, basically, the two switches will “help” each other to synchronize…this might be useful in case the CoreRouter fails someday:

<pre><code><strong>SW1(config)#ntp peer 192.168.123.2
</strong></code></pre>

<pre><code><strong>SW2(config)#ntp peer 192.168.123.1
</strong></code></pre>

After waiting a few minutes, you’ll see that SW1 and SW2 have synchronized with each other:

<pre><code><strong>SW1#show ntp associations
</strong>
      address         ref clock     st  when  poll reach  delay  offset    disp
*~192.168.123.3    146.185.130.223   3    59    64   37     3.0   -0.74   877.4
+~192.168.123.2    192.168.123.3     4    50   128  376     2.2   -2.04     1.3
 * master (synced), # master (unsynced), + selected, - candidate, ~ configured
</code></pre>

<pre><code><strong>SW2#show ntp associations
</strong>
      address         ref clock     st  when  poll reach  delay  offset    disp
*~192.168.123.3    146.185.130.223   3    45   128  377     2.9    1.95     1.0
 ~192.168.123.1    192.168.123.3     4    67  1024  376     1.8    2.40     1.4
 * master (synced), # master (unsynced), + selected, - candidate, ~ configured
</code></pre>

Great, everything is now in sync.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here, you will find the final configuration of each device.
{% endtab %}

{% tab title="CoreRouter" %}
```
hostname CoreRouter
!
interface FastEthernet0/0
 ip address 192.168.123.3 255.255.255.0
!
ip name-server 8.8.8.8
!
ntp server pool.ntp.org
ntp update-calendar
!
end
```
{% endtab %}

{% tab title="SW1" %}
```
hostname SW1
!
interface FastEthernet0/24
 ip address 192.168.123.1 255.255.255.0
!
ntp server 192.168.123.3
ntp peer 192.168.123.2
!
end
```
{% endtab %}

{% tab title="SW2" %}
```
hostname SW2
!
interface FastEthernet0/24
 ip address 192.168.123.2 255.255.255.0
!
ntp server 192.168.123.3
ntp peer 192.168.123.1
!
end
```
{% endtab %}
{% endtabs %}

Are we done? Not quite yet…there are a few more things we can do with NTP. The CoreRouter and the two switches use unicast (UDP port 123) for synchronization, but you can also use multicast or broadcast. Let me give you an example…

#### Multicast and Broadcast

If you have more than 20 network devices or a router that has limited system memory or CPU resources, you might want to consider using NTP broadcast or multicast as it requires fewer resources. We can enable multicast or broadcast on the interface level. To demonstrate this, I will add two routers below SW1 and SW2 that will synchronize themselves using multicast or broadcast. This is what it looks like:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/07/cisco-ntp-broadcast-multicast-example-topology-1.png" alt=""><figcaption></figcaption></figure>

I’ll configure SW1 to use multicast address 239.1.1.1, and SW2 will send NTP updates through broadcast:

<pre><code><strong>SW1(config)#interface vlan 10
</strong><strong>SW1(config-if)#ntp multicast 239.1.1.1
</strong></code></pre>

<pre><code><strong>SW2(config-if)#interface vlan 20 
</strong><strong>SW2(config-if)#ntp broadcast
</strong></code></pre>

R5 will synchronize itself by using multicast:

<pre><code><strong>R5(config)#interface fastEthernet 0/0
</strong><strong>R5(config-if)#ntp multicast client 239.1.1.1
</strong></code></pre>

The commands are pretty self-explanatory, let’s see if it worked:

<pre><code><strong>R5#show ntp associations 
</strong>
  address         ref clock       st   when   poll reach  delay  offset   disp
* 192.168.10.1    192.168.123.3    4     14     64     1  1.528  -1.209  0.206
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
</code></pre>

<pre><code><strong>R5#show ntp status 
</strong>Clock is synchronized, stratum 5, reference is 192.168.10.1  
nominal freq is 250.0000 Hz, actual freq is 250.0174 Hz, precision is 2**24
reference time is D765447B.DA56D83C (16:08:27.852 UTC Mon Jul 7 2014)
clock offset is -0.0012 msec, root delay is 0.01 msec
root dispersion is 0.16 msec, peer dispersion is 0.00 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is -0.000069583 s/s
system poll interval is 64, last update was 35 sec ago.
</code></pre>

You can see that it has synchronized itself, and it shows the IP address of SW1. Let’s see if we can get broadcast to work on R6:

<pre><code><strong>R6(config)#interface fastEthernet 0/0
</strong><strong>R6(config-if)#ntp broadcast client 
</strong></code></pre>

<pre><code><strong>R6#show ntp associations 
</strong>
  address         ref clock       st   when   poll reach  delay  offset   disp
* 192.168.20.2    192.168.123.3    4     29     64     1  1.284  -4.035  0.127
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
</code></pre>

<pre><code><strong>R6#show ntp status 
</strong>Clock is synchronized, stratum 5, reference is 192.168.20.2  
nominal freq is 250.0000 Hz, actual freq is 250.0132 Hz, precision is 2**24
reference time is D7654496.15979782 (16:08:54.084 UTC Mon Jul 7 2014)
clock offset is -0.0040 msec, root delay is 0.01 msec
root dispersion is 0.59 msec, peer dispersion is 0.00 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is -0.000052939 s/s
system poll interval is 64, last update was 29 sec ago.
</code></pre>

Excellent! Two more network devices that are synchronized.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here, you will find the startup configuration of each device.
{% endtab %}

{% tab title="R5" %}
```
hostname R5
!
interface FastEthernet0/0
 ntp multicast client 239.1.1.1
!
end
```
{% endtab %}

{% tab title="R6" %}
```
hostname R6
!
interface FastEthernet0/0
 ntp broadcast client 
!
end
```
{% endtab %}

{% tab title="SW1" %}
```
hostname SW1
!
interface FastEthernet0/24
 ip address 192.168.123.1 255.255.255.0
!
interface vlan 10
 ntp multicast 239.1.1.1
!
end
```
{% endtab %}

{% tab title="SW2" %}
```
hostname SW2
!
interface FastEthernet0/24
 ip address 192.168.123.2 255.255.255.0
!
interface vlan 20 
 ntp broadcast
!
end
```
{% endtab %}
{% endtabs %}

You now know how to configure NTP, but there is one more important topic to cover…security! Right now, our routers will accept any source as the NTP server, and they will serve any NTP client that requests updates. To protect our network, we will have to configure authentication and access control. Let’s start with authentication.

#### Authentication

When we enable authentication, all NTP packets that can update the clock have to be authenticated. The packets will be authenticated using HMAC MD5, which carries a key number.

I want to make sure that SW1 and SW2 will authenticate the CoreRouter so they don’t just accept any NTP updates from a device that has IP address 192.168.123.3 configured. We’ll configure the router first:

<pre><code><strong>CoreRouter(config)#ntp authenticate
</strong><strong>CoreRouter(config)#ntp trusted-key 1
</strong><strong>CoreRouter(config)#ntp trusted-key 2
</strong><strong>CoreRouter(config)#ntp authentication-key 1 md5 NETWORKLESSONS1
</strong><strong>CoreRouter(config)#ntp authentication-key 2 md5 NETWORKLESSONS2
</strong></code></pre>

Each switch will use a different key for authentication. The **ntp authentication-key** command is required to set the key number and the password. The **ntp trusted-key** command is a bit weird, if you don’t use it then the key that you configured will not be activated so don’t forget it.

Let’s configure the switches now:

<pre><code><strong>SW1(config)#ntp authenticate 
</strong><strong>SW1(config)#ntp authentication-key 1 md5 NETWORKLESSONS1
</strong><strong>SW1(config)#ntp trusted-key 1
</strong><strong>SW1(config)#ntp server 192.168.123.3 key 1
</strong></code></pre>

<pre><code><strong>SW2(config)#ntp authenticate
</strong><strong>SW2(config)#ntp authentication-key 2 md5 NETWORKLESSONS2
</strong><strong>SW2(config)#ntp trusted-key 2
</strong><strong>SW2(config)#ntp server 192.168.123.3 key 2
</strong></code></pre>

The configuration on the switches is similar, but the difference is that we also specified the key for the NTP server. SW1 and SW2 will only use 192.168.123.3 to synchronize their clocks if the MD5 signature is correct.

Earlier, we configured SW1 and SW2 to use each other as peers, and of course, we can also use authentication for this. It looks like this:

<pre><code><strong>SW1(config)#ntp authentication-key 12 md5 NETWORKLESSONS12
</strong><strong>SW1(config)#ntp trusted-key 12
</strong><strong>SW1(config)#ntp peer 192.168.123.2 key 12 
</strong></code></pre>

<pre><code><strong>SW2(config)#ntp authentication-key 12 md5 NETWORKLESSONS12                     
</strong><strong>SW2(config)#ntp trusted-key 12                                                 
</strong><strong>SW2(config)#ntp peer 192.168.123.1 key 12
</strong></code></pre>

The configuration is similar: configure a key and make it trusted. We change the NTP peer command so that it requires authentication.

Authentication is great, but there is still one security problem to tackle. A NTP server will serve updates to _any_ NTP client, and a NTP client will accept _any_ IP address as the NTP server. To solve this, we can implement some access-list…let’s take a look!

#### Access-Control

First, I will configure the CoreRouter to only accept one IP address as its NTP server. This is tricky since the IP address might change in the future. If you implement this on a production network, you’ll have to make sure that you add all the possible IP addresses in the access-list:

<pre><code><strong>CoreRouter(config)#access-list 1 permit 146.185.130.223
</strong><strong>CoreRouter(config)#ntp access-group peer 1
</strong></code></pre>

The IP address above is what pool.ntp.org resolves to for me. The **`ntp access-group peer`** command is used to activate the access-list. SW1 and SW2 are the NTP clients for the CoreRouter, but right now, everyone can use our router as the NTP server. Let’s fix this so only SW1 and SW2 are allowed as NTP clients:

<pre><code><strong>CoreRouter(config)#ntp access-group serve-only 12
</strong><strong>CoreRouter(config)#access-list 12 permit 192.168.123.1
</strong><strong>CoreRouter(config)#access-list 12 permit 192.168.123.2
</strong></code></pre>

Problem solved. Only SW1 and SW2 are now accepted as NTP clients. Our CoreRouter is now protected, but let’s make some changes on SW1 and SW2 as well:

<pre><code><strong>SW1(config)#access-list 3 permit 192.168.123.2
</strong><strong>SW1(config)#access-list 3 permit 192.168.123.3
</strong><strong>SW1(config)#ntp access-group peer 3
</strong></code></pre>

<pre><code><strong>SW2(config)#access-list 3 permit 192.168.123.1
</strong><strong>SW2(config)#access-list 3 permit 192.168.123.3
</strong><strong>SW2(config)#ntp access-group peer 3
</strong></code></pre>

The configuration above allows SW1 and SW2 to use CoreRouter and each other as NTP servers, no other sources are allowed.

{% hint style="info" %}
In my example, I used a public server for NTP (pool.ntp.org), but you can also configure a router or switch as the NTP master and set a stratum number yourself. If you do this, you’ll need the `ntp master`command, and your device will synchronize its clock using the 127.127.7.1 or 127.127.1.1 IP address. Make sure you permit this IP address in your access-list!
{% endhint %}

After we configure authentication we can verify if it’s working or not. Here’s an example for SW1:

<pre><code><strong>SW1#show ntp associations detail 
</strong>192.168.123.3 configured, authenticated, our_master, sane, valid, stratum 3
ref ID 146.185.130.223, time D7656103.9F50193C (18:10:11.622 UTC Mon Jul 7 2014)
our mode client, peer mode server, our poll intvl 1024, peer poll intvl 1024
root delay 12.28 msec, root disp 144.93, reach 377, sync dist 162.231
delay 1.43 msec, offset -0.7465 msec, dispersion 1.48
precision 2**24, version 3
org time D76562BA.DE84436E (18:17:30.869 UTC Mon Jul 7 2014)
rcv time D76562BA.DEE4A769 (18:17:30.870 UTC Mon Jul 7 2014)
xmt time D76562BA.DE7AA858 (18:17:30.869 UTC Mon Jul 7 2014)
filtdelay =     1.43    1.39    1.37    1.40    1.51    2.43    1.50    1.63
filtoffset =   -0.75   -0.79   -3.92   -4.03   -3.95   -1.69   -1.23   -1.70
filterror =     0.02   15.64   31.27   46.89   62.52   78.14   79.12   79.74

192.168.123.2 configured, authenticated, selected, sane, valid, stratum 4
ref ID 192.168.123.3, time D76561FF.17C145F3 (18:14:23.092 UTC Mon Jul 7 2014)
our mode active, peer mode active, our poll intvl 1024, peer poll intvl 1024
root delay 14.97 msec, root disp 149.15, reach 377, sync dist 164.719
delay 4.21 msec, offset -7.8822 msec, dispersion 5.98
precision 2**18, version 3
org time D7656506.196F1052 (18:27:18.099 UTC Mon Jul 7 2014)
rcv time D7656506.1BFDEA18 (18:27:18.109 UTC Mon Jul 7 2014)
xmt time D7656486.DED15803 (18:25:10.870 UTC Mon Jul 7 2014)
filtdelay =     4.21    1.63    2.76    4.39    4.73    1.27    1.14    1.17
filtoffset =   -7.88   -4.20   -4.28   -3.79   -2.31   -0.20    0.03    0.29
filterror =     1.97   17.59   18.57   19.55   35.17   50.80   66.42   82.05
</code></pre>

That’s all there is to it. Hopefully, this NTP lesson is helpful for you to understand and configure NTP in your network.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here, you will find the startup configuration of each device.
{% endtab %}

{% tab title="CoreRouter" %}
```
hostname CoreRouter
!
interface FastEthernet0/0
 ip address 192.168.123.3 255.255.255.0
!
ip name-server 8.8.8.8
ntp server pool.ntp.org
ntp update-calendar
ntp authenticate
ntp trusted-key 1
ntp trusted-key 2
ntp authentication-key 1 md5 NETWORKLESSONS1
ntp authentication-key 2 md5 NETWORKLESSONS2
!
access-list 1 permit 146.185.130.223
ntp access-group peer 1
ntp access-group serve-only 12
access-list 12 permit 192.168.123.1
access-list 12 permit 192.168.123.2
!
end
```
{% endtab %}

{% tab title="SW1" %}
```
hostname SW1
!
interface FastEthernet0/24
 ip address 192.168.123.1 255.255.255.0
!
ntp authenticate 
ntp authentication-key 1 md5 NETWORKLESSONS1
ntp trusted-key 1
ntp server 192.168.123.3 key 1
!
ntp authentication-key 12 md5 NETWORKLESSONS12
ntp trusted-key 12
ntp peer 192.168.123.2 key 12 
!
access-list 3 permit 192.168.123.2
access-list 3 permit 192.168.123.3
ntp access-group peer 3
!
end
```
{% endtab %}

{% tab title="SW2" %}
```
hostname SW2
!
interface FastEthernet0/24
 ip address 192.168.123.2 255.255.255.0
!
ntp authenticate
ntp authentication-key 2 md5 NETWORKLESSONS2
ntp trusted-key 2
ntp server 192.168.123.3 key 2
!
ntp authentication-key 12 md5 NETWORKLESSONS12                     
ntp trusted-key 12                                                 
ntp peer 192.168.123.1 key 12
!
access-list 3 permit 192.168.123.1
access-list 3 permit 192.168.123.3
ntp access-group peer 3
!
end
```
{% endtab %}
{% endtabs %}

If you enjoyed reading this, please share it with your friends/colleagues or leave a comment if you have any questions!
