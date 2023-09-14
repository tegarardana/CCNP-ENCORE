# Troubleshooting NTP

In this lesson we’ll take a look at troubleshooting NTP (Network Time Protocol). NTP is an important protocol to use since it ensures our logging information has the correct time and date.

There are a couple of things that could go wrong with NTP:

* **NTP traffic filtered**: access-lists could block NTP traffic.
* **NTP Authentication issues**: NTP supports authentication, client and server need to use the same settings.
* **Time offset too high**: When the time offset between client/server is too large it will take a very long time to synchronize.
* **Stratum level too high**: The stratum level is between 1 (best) and 15 (worst). A stratum level of 16 is considered unusable.
* **NTP server source filter**: NTP servers can be configured to allow only clients from certain IP addresses.

We’ll take a look at these different issues. I will use the following two routers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2015/04/ntp-client-server-r1-r2.png" alt=""><figcaption></figcaption></figure>

R1 will be our NTP client and R2 will be the NTP server. There are two useful commands that we should start with:

<pre><code><strong>R1#show ntp status
</strong>Clock is unsynchronized, stratum 16, no reference clock
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**18
reference time is 00000000.00000000 (00:00:00.000 UTC Mon Jan 1 1900)
clock offset is 0.0000 msec, root delay is 0.00 msec
root dispersion is 0.00 msec, peer dispersion is 0.00 msec
</code></pre>

<pre><code><strong>R1#show ntp associations
</strong>
      address         ref clock     st  when  poll reach  delay  offset    disp
 ~192.168.12.2     0.0.0.0          16     -    64    0     0.0    0.00  16000.
 * master (synced), # master (unsynced), + selected, - candidate, ~ configured
</code></pre>

These tell us that R1 has 192.168.12.2 configured as the NTP server and it’s currently not synchronized. Let’s check if R1 is receiving NTP packets, we can use a debug for this:

<pre><code><strong>R1#debug ntp packets
</strong>NTP packets debugging is on

NTP: xmit packet to 192.168.12.2:
 leap 3, mode 3, version 3, stratum 0, ppoll 64
 rtdel 0000 (0.000), rtdsp 10001 (1000.015), refid 00000000 (0.0.0.0)
 ref 00000000.00000000 (00:00:00.000 UTC Mon Jan 1 1900)
 org 00000000.00000000 (00:00:00.000 UTC Mon Jan 1 1900)
 rec 00000000.00000000 (00:00:00.000 UTC Mon Jan 1 1900)
 xmt C0295220.A6D7FB14 (01:04:32.651 UTC Fri Mar 1 2002)
</code></pre>

This debug tells us that R1 is sending NTP packets but we are not receiving anything from the NTP server. Make sure NTP server is allowed to go through:

<pre><code><strong>R1#show access-lists
</strong>Extended IP access list NO_TIME
    1 deny udp any any eq ntp (3 matches)
</code></pre>

R1 uses UDP port 123 so make sure it’s not blocked, let’s get rid of this access-list:

<pre><code><strong>R1(config)#interface FastEthernet 0/0
</strong><strong>R1(config-if)#no ip access-group NO_TIME in
</strong></code></pre>

After removing the access-list, NTP will be able to use the NTP packets from the server:

```
R1#
NTP: rcv packet from 192.168.12.2 to 192.168.12.1 on FastEthernet0/0:
 leap 0, mode 4, version 3, stratum 1, ppoll 64
 rtdel 0000 (0.000), rtdsp 0002 (0.031), refid 4C4F434C (76.79.67.76)
 ref C02952FA.C3830714 (01:08:10.763 UTC Fri Mar 1 2002)
 org C0295320.A6D6D314 (01:08:48.651 UTC Fri Mar 1 2002)
```

Here’s the end result:

<pre><code><strong>R1#show ntp status
</strong>Clock is synchronized, stratum 2, reference is 192.168.12.2
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**18
reference time is C0295360.FAE093C8 (01:09:52.979 UTC Fri Mar 1 2002)
clock offset is 3.9974 msec, root delay is 47.78 msec
root dispersion is 15879.03 msec, peer dispersion is 15875.02 msec
</code></pre>

The clock is now synchronized. Another issue you can spot with the debugging NTP is an authentication mismatch:

<pre><code><strong>R1(config)#ntp server 192.168.12.2 key 1
</strong><strong>R1(config)#ntp authentication-key 1 md5 MY_KEY
</strong></code></pre>

I’ll configure R1 to only accepts NTP packets from the NTP server that are authenticated with a certain key. The NTP server however doesn’t use any form of authentication. We can find this error with the following debug:

<pre><code><strong>R1#debug ntp validity
</strong>NTP peer validity debugging is on
</code></pre>

It will tell us this:

```
R1#
NTP: packet from 192.168.12.2 failed validity tests 10
Authentication failed
```

Make sure your NTP authentication settings match on both sides.

When the time/date difference between the NTP server and NTP client is significant, it will take a long time to sync. Right now the clocks look like this:

<pre><code><strong>R1#show clock
</strong>.01:32:25.208 UTC Fri Mar 1 2002
</code></pre>

<pre><code><strong>R2#show clock
</strong>18:01:44.327 UTC Fri Jan 30 2015
</code></pre>

Setting the clock on the NTP client to something close to the NTP server will speed up the sync process a lot:

<pre><code><strong>R1#clock set 18:00:00 30 January 2015
</strong></code></pre>

In a few minutes, the clock on the NTP client should be synced.

Another issue with NTP is that the stratum level is limited, we can use values between 1 (best) and 15 (worst). If the NTP server has a stratum level of 15 then the NTP client won’t be able to sync since 16 is considered unreachable.

Debugging the NTP packets on the client will reveal this:

```
R1#
NTP: rcv packet from 192.168.12.2 to 192.168.12.1 on FastEthernet0/0:
 leap 0, mode 4, version 3, stratum 15, ppoll 1024
 rtdel 0000 (0.000), rtdsp 0002 (0.031), refid 7F7F0701 (127.127.7.1)
 ref D876471C.52EF5E3C (18:08:28.323 UTC Fri Jan 30 2015)
 org D876474C.7D0383EE (18:09:16.488 UTC Fri Jan 30 2015)
 rec D876474C.80123528 (18:09:16.500 UTC Fri Jan 30 2015)
 xmt D876474C.8312ABB8 (18:09:16.512 UTC Fri Jan 30 2015)
 inp D876474C.87424408 (18:09:16.528 UTC Fri Jan 30 2015)
```

R1 will never be able to synchronize itself since the NTP server is announcing itself as stratum 15. You can fix this by setting a lower NTP stratum value on your NTP server:

<pre><code><strong>R2(config)#ntp master 1
</strong></code></pre>

I’ll change it to stratum value 1. This allows R1 to synchronize itself:

<pre><code><strong>R1#show ntp status
</strong>Clock is synchronized, stratum 2, reference is 192.168.12.2
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**18
reference time is C0295360.FAE093C8 (01:09:52.979 UTC Fri Mar 1 2002)
clock offset is 3.9974 msec, root delay is 47.78 msec
root dispersion is 15879.03 msec, peer dispersion is 15875.02 msec
</code></pre>

Last but not least, NTP servers can be configured to only allow NTP clients from certain IP addresses:

<pre><code><strong>R2(config)#ntp access-group ?
</strong>  peer        Provide full access
  query-only  Allow only control queries
  serve       Provide server and query access
  serve-only  Provide only server access
</code></pre>

For example I’ll configure it to only allow IP address 1.1.1.1:

<pre><code><strong>R2(config)#ntp access-group serve 1
</strong><strong>R2(config)#access-list 1 permit 1.1.1.1
</strong><strong>R2(config)#ip route 1.1.1.1 255.255.255.255 192.168.12.1
</strong></code></pre>

In this case, I need to make sure that the NTP client is sourcing its NTP packets from the correct IP address:

<pre><code><strong>R1(config)#interface loopback 0
</strong><strong>R1(config-if)#ip address 1.1.1.1 255.255.255.255
</strong></code></pre>

<pre><code><strong>R1(config)#ntp source loopback0
</strong></code></pre>

The NTP source command will tell R1 to use IP address 1.1.1.1 from its loopback interface as the source of its NTP packets.

That’s pretty much it, these are the most common NTP issues that you might encounter on Cisco IOS devices. If you have any questions, feel free to leave a comment!
