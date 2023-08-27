# IP SLA EEM Script

In our [IP SLA lesson](https://networklessons.com/cisco/ccnp-encor-350-401/ip-sla-service-level-agreement-on-cisco-ios) we explained how you can “measure” network performance by sending “probes” to remote devices. We also talked about[ EEM (Embedded Event Manager)](https://networklessons.com/cisco/ccnp-encor-350-401/cisco-ios-embedded-event-manager) which we can use for scripting on our IOS devices.

In this lesson we’ll take a look how to combine IP SLA and EEM. This can be useful as it allows you to perform certain actions when IP SLA reports a failure. For example, we can use this to produce custom syslog messages and send emails to the administrator.

Here’s the topology we will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/02/r1-r2-gigabit-links.png" alt=""><figcaption></figcaption></figure>

We only need two routers to demonstrate this. IP SLA is configured on R1 which sends ICMP echoes to R2.

## Configuration

Here’s what the IP SLA configuration looks like:

<pre><code><strong>R1#show running-config | begin ip sla
</strong>ip sla 1
 icmp-echo 192.168.12.2
 frequency 10
ip sla schedule 1 life forever start-time now
</code></pre>

It’s a simple configuration where R1 will keep sending ICMP echoes to R2 forever. To combine IP SLA with EEM, we’ll need to track it somehow. We can do this with object tracking:

<pre><code><strong>R1(config)#track 1 ip sla 1 reachability 
</strong></code></pre>

Above we created a new object that will track IP SLA 1. We can now track the status of this object with EEM:

<pre><code><strong>R1(config)#event manager applet TRACK_IP_DOWN
</strong><strong>R1(config-applet)#event track 1 state down
</strong><strong>R1(config-applet)#action 1.0 syslog msg "IP SLA 1 is down"
</strong><strong>R1(config-applet)#action 2.0 mail server "smtp.mailserver.local" to "support@networklessons.com" from "support@networklessons.com" subject "IP SLA 1 is down" body "IP SLA 1 is not receiving ICMP echo replies anymore"
</strong></code></pre>

As soon as the object goes down, EEM will perform two actions:

* We produce a syslos message which says “IP SLA 1 is down”.
* We send an e-mail to e-mail server “smtp.mailserver.local” using the email addresses, subject and body that I specified above.

We’ll also configure an action that will be performed when the object is up again:

<pre><code><strong>R1(config)#event manager applet IP_SLA_1_UP
</strong><strong>R1(config-applet)#event track 1 state up
</strong><strong>R1(config-applet)#action 1.0 syslog msg "IP SLA 1 is up"
</strong></code></pre>

Once the object is up, we will generate a syslog message. Let’s verify our work…

## Verification

To test our work we need to trigger a failure. When our IP SLA ICMP echoes are replied, the “successes” counter will increase. When we don’t get a reply to our ICMP echoes then the “failures” counter will increase:

<pre><code><strong>R1#show ip sla statistics 
</strong>IPSLAs Latest Operation Statistics

IPSLA operation id: 1
        Latest RTT: 3 milliseconds
Latest operation start time: 10:16:41 UTC Thu Feb 18 2016
Latest operation return code: OK
Number of successes: 56
Number of failures: 0
Operation time to live: Forever
</code></pre>

The most simple way to simulate a failure is to shut one of the interfaces. You can also configure IP SLA so that it will trigger a failure when certain thresholds are exceeded (for example when the RTT exceeds a certain value).

I’ll shut one of the interfaces but before we do this, let’s enable some EEM debugging:

<pre><code><strong>R1#debug event manager action cli
</strong>Debug EEM action cli debugging is on
</code></pre>

<pre><code><strong>R1#debug event manager action mail
</strong>Debug EEM action mail debugging is on
</code></pre>

Now we will shut the interface on R2:

<pre><code><strong>R2(config)#interface GigabitEthernet 0/1
</strong><strong>R2(config-if)#shutdown
</strong></code></pre>

Here’s what happens on R1:

```
R1#
%TRACK-6-STATE: 1 ip sla 1 reachability Up -> Down
%HA_EM-6-LOG: IP_SLA_1_DOWN: IP SLA 1 is down
```

The first message is produced by object tracking. It notices that IP SLA has reported a failure. The second message is produced by EEM and it’s the first action that we configured, the syslog message.

Here’s the second EEM action:

```
R1#
%HA_EM-6-LOG: fh_send_mail:  : DEBUG(smtp_lib) : &lt;?xml version="1.0" encoding="UTF-8" ?&gt;&lt;fh_smtp_args&gt;&lt;fh_smtp_port&gt;25&lt;/fh_smtp_port&gt;&lt;fh_smtp_secure&gt;0&lt;/fh_smtp_secure&gt;&lt;/fh_smtp_args&gt;
%HA_EM-6-LOG: IP_SLA_1_DOWN : DEBUG(smtp_lib) : smtp_connect_attempt: 1
```

Above you can see that EEM is attempting to send the email. I don’t have any mailservers that are reachable but this proves that the EEM action is working.

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you will find the final configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
track 1 ip sla 1 reachability
!
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
ip sla 1
 icmp-echo 192.168.12.2
 frequency 10
ip sla schedule 1 life forever start-time now
!
event manager applet IP_SLA_1_DOWN
 event track 1 state down
 action 1.0 syslog msg "IP SLA 1 is down"
 action 2.0 mail server "smtp.mailserver.local" to "support@networklessons.com" from "support@networklessons.com" subject "IP SLA 1 is down" body "IP SLA 1 is not receiving ICMP echo replies anymore"
event manager applet IP_SLA_1_UP
 event track 1 state up
 action 1.0 syslog msg "IP SLA 1 is up"
!
end
```
{% endtab %}

{% tab title="R2" %}
```
hostname R2
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
control-plane
!
end
```
{% endtab %}
{% endtabs %}

## Conclusion

Combining IP SLA and EEM works very well and it might be a wise idea to implement this. With the actions that EEM offers we can be notified immediately when IP SLA is having any issues. I hope this has been useful, if you have any questions feel free to leave a comment in our forum.
