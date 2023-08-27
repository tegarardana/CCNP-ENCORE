# Embedded Event Manager (EEM)

Embedded Event Manager (EEM) is a technology on Cisco Routers that lets you run scripts or commands when a certain event happens. It’s probably best just to show you some examples to see how it works. This is the topology that I will use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/05/R1-R2.png" alt=""><figcaption></figcaption></figure>

## Syslog Events

Syslog messages are the messages that you see by default on your console. Interfaces going up or down, OSPF neighbors that dissapear and such are all syslog messages. EEM can take action when one of these messages show up. Let’s start with an example that enables an interface once it goes down.

### Interface Recovery

```
R2(config)#
event manager applet INTERFACE_DOWN 
 event syslog pattern "Interface FastEthernet0/0, changed state to down"
 action 1.0 cli command "enable"
 action 2.0 cli command "conf term"
 action 3.0 cli command "interface fa0/0"
 action 4.0 cli command "no shut"
```

The applet is called “INTERFACE\_DOWN” and the event is a syslog pattern that matches the text when an interface goes down. When this occurs, we run a number of commands. What happens is that whenever someone shuts the interface, EEM will do a “no shut” on it.

To demonstrate that this works I’ll enable a debug:

<pre><code><strong>R2#debug event manager action cli
</strong>Debug EEM action cli debugging is on
</code></pre>

This will show the commands that EEM runs when the event occurs. Let’s do a shut on that interface:

<pre><code><strong>R2(config)#interface FastEthernet 0/0
</strong><strong>R2(config-if)#shutdown
</strong></code></pre>

Within a few seconds you will see this:

```
R2#
%LINK-5-CHANGED: Interface FastEthernet0/0, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to down

%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : CTL : cli_open called.
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : OUT : R2>
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : IN  : R2>enable
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : OUT : R2#
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : IN  : R2#conf term
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : OUT : Enter configuration commands, one per line.  End with CNTL/Z.
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : OUT : R2(config)#
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : IN  : R2(config)#interface fa0/0
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : OUT : R2(config-if)#
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : IN  : R2(config-if)#no shut
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : OUT : R2(config-if)#
%HA_EM-6-LOG: INTERFACE_DOWN : DEBUG(cli_lib) : : CTL : cli_close called.

%LINK-3-UPDOWN: Interface FastEthernet0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up
```

The interface went down, EEM runs the commands and the interface is up again. Simple but I think this is a good example to demonstrate how EEM works. Let’s see what else we can do…

### OSPF Adjacency Changes

The next example is perhaps useful. Whenever the OSPF adjacency dissapears you will see a syslog message on your console. We’ll use this message as the event and once it occurs, we enable OSPF adjacency debugging and send an e-mail:

```
R2(config)#
event manager applet OSPF_DOWN 
 event syslog pattern "Nbr 192.168.12.1 on FastEthernet0/0 from FULL to DOWN"
 action 1.0 cli command "enable"
 action 2.0 cli command "debug ip ospf adj"
 action 3.0 mail server "smtp.ziggo.nl" to "info@networklessons.com" from "R2@networklessons.com" subject "OSPF IS DOWN" body "Please fix OSPF"
```

The event that I used is a syslog message that should look familiar. The first two actions are executed on the CLI but the third action is for the e-mail. It will send a message to info@networklessons.com through SMTP-server “smtp.ziggo.nl”.

Let’s give it a try. I have to enable another debug if I want to see the mail action:

<pre><code><strong>R2#debug event manager action mail 
</strong>Debug EEM action mail debugging is on
</code></pre>

Once the OSPF neighbor adjacency is established, I’ll shut the interface on one of the routers so it breaks:

<pre><code><strong>R1(config)#interface FastEthernet 0/0
</strong><strong>R1(config-if)#shutdown
</strong></code></pre>

And this is what you’ll see:

```
R2#
Translating "smtp.ziggo.nl"...domain server (255.255.255.255)

%OSPF-5-ADJCHG: Process 1, Nbr 192.168.12.1 on FastEthernet0/0 from FULL to DOWN, Neighbor Down: Dead timer expired

%HA_EM-6-LOG: OSPF_DOWN : DEBUG(cli_lib) : : CTL : cli_open called.
%HA_EM-6-LOG: OSPF_DOWN : DEBUG(cli_lib) : : OUT : R2>
%HA_EM-6-LOG: OSPF_DOWN : DEBUG(cli_lib) : : IN  : R2>enable
%HA_EM-6-LOG: OSPF_DOWN : DEBUG(cli_lib) : : OUT : R2#
%HA_EM-6-LOG: OSPF_DOWN : DEBUG(cli_lib) : : IN  : R2#debug ip ospf adj
%HA_EM-6-LOG: OSPF_DOWN : DEBUG(cli_lib) : : OUT : OSPF adjacency events debugging is on
%HA_EM-6-LOG: OSPF_DOWN : DEBUG(cli_lib) : : OUT : R2#
%HA_EM-6-LOG: OSPF_DOWN : DEBUG(smtp_lib) : smtp_connect_attempt: 1

OSPF: Build router LSA for area 0, router ID 192.168.12.2, seq 0x8000000B, process 1
OSPF: No full nbrs to build Net Lsa for interface FastEthernet0/0
OSPF: Build network LSA for FastEthernet0/0, router ID 192.168.12.2
OSPF: Build network LSA for FastEthernet0/0, router ID 192.168.12.2

%HA_EM-6-LOG: OSPF_DOWN : DEBUG(smtp_lib) : fh_smtp_connect failed at attempt 1
Translating "smtp.ziggo.nl"...domain server (255.255.255.255)
%HA_EM-6-LOG: OSPF_DOWN : DEBUG(smtp_lib) : smtp_connect_attempt: 2

%HA_EM-6-LOG: OSPF_DOWN : DEBUG(smtp_lib) : fh_smtp_connect callback timer is awake
%HA_EM-3-FMPD_SMTP: Error occurred when sending mail to SMTP server: smtp.ziggo.nl : timeout error
%HA_EM-6-LOG: OSPF_DOWN : DEBUG(cli_lib) : : CTL : cli_close called.
```

My router isn’t connected to the Internet but you can see it’s trying to contact the SMTP server and send an e-mail. It also enabled the OSPF adjacency debug thanks to the CLI commands.

## CLI Events

The previous two examples used syslog messages as the event but you can also take action based on commands that are used on the CLI. The example below is a funny one, whenever someone watches the running-configuration it will exclude all lines with the word “interface” in it:

```
R2(config)#
event manager applet SHOW_RUN_NO_INTERFACES 
 event cli pattern "show run" sync yes
 action 1.0 cli command "enable"
 action 2.0 cli command "show run | exclude interface"
 action 3.0 puts "$_cli_result"
 action 4.0 set $_exit_status "0"
```

As you can see above the event is a CLI pattern. the “sync yes” parameter is required, this tells EEM to run the script before running the “show run” command. When the script is done, it sets the exit status to 0. Basically this means that whenever someone uses the “show run” command, the script will run “show run | exclude interface” instead and gives you the output.

Let’s see what the result is…

<pre><code><strong>R2#show running-config 
</strong>
Building configuration...
</code></pre>

You will see the output of the running configuration and if you left the debug on, you’ll see what EEM is doing behind the scenes:

```
R2#
%HA_EM-6-LOG: SHOW_RUN_NO_INTERFACES : DEBUG(cli_lib) : : CTL : cli_open called.
%HA_EM-6-LOG: SHOW_RUN_NO_INTERFACES : DEBUG(cli_lib) : : OUT : R2>
%HA_EM-6-LOG: SHOW_RUN_NO_INTERFACES : DEBUG(cli_lib) : : IN  : R2>enable
%HA_EM-6-LOG: SHOW_RUN_NO_INTERFACES : DEBUG(cli_lib) : : OUT : R2#
%HA_EM-6-LOG: SHOW_RUN_NO_INTERFACES : DEBUG(cli_lib) : : IN  : R2#show run | exclude interface
%HA_EM-6-LOG: SHOW_RUN_NO_INTERFACES : DEBUG(cli_lib) : : OUT : Building configuration...
```

Somewhere further down the running-config you can see that the lines with “interface” in them were removed:

```
!
 ip address 192.168.12.2 255.255.255.0
 duplex auto
 speed auto
```

While this isn’t very useful, I think this is a good example to see what it does. A good real life scenario might be hiding all lines that have “username” or “enable secret” in them for certain users.

## Interface Events

You have seen syslog and CLI pattern events, but we have some others. What about interface counters? It might be useful to perform an action when some interface counters have a certain value. Here’s an example:

<pre><code><strong>R2#show interfaces fastEthernet 0/0 | incl load
</strong>     reliability 255/255, txload 1/255, rxload 1/255
</code></pre>

Let’s create a script that does something when the interface load hits a certain value. To make this work, it’s best to change the load interval of the interface first:

<pre><code><strong>R2(config)#interface FastEthernet 0/0
</strong><strong>R2(config-if)#load-interval 30
</strong></code></pre>

By using this command, the router will calculate the load of the interface every 30 seconds, the default is 5 minutes. Let’s create the script:

```
R2(config)#
event manager applet INTERFACE_LOAD 
 event interface name FastEthernet0/0 parameter rxload entry-op gt entry-val 10 entry-type value poll-interval 10
 action 1.0 syslog priority informational msg "INTERFACE OVERLOADED"
```

This event is a bit harder to read…when the rx load of the interface is above 10/255 then we will take action. Every 10 seconds we will check if we reached this value or not. When the event occurs, a syslog message is produced.

To demonstrate this we’ll send some packets from R1 towards R2:

<pre><code><strong>R1#ping 192.168.12.2 repeat 9999999 size 15000 timeout 0
</strong></code></pre>

Once the interface rx load is above 10 you’ll see the following message on the console:

```
R2#
%HA_EM-6-LOG: INTERFACE_LOAD: INTERFACE OVERLOADED
```

Pretty neat right? Sending an e-mail as the action might be a good idea when the interface load is above 60-70%.

## Scheduling Events

Instead of launching actions based on syslog or CLI messages we can also use scheduled tasks. This means that you can run actions every X minutes / hours / days etc. Here’s an example:

```
R2(config)#
event manager applet TIMER 
 event timer watchdog time 60
 action 1.0 cli command "enable"
 action 2.0 cli command "write memory"
 action 3.0 syslog priority informational msg "Configuration has been saved"
```

This script runs every 60 seconds and runs the “write memory” command. Once it’s done, it will produce a syslog message. After waiting for 60 seconds we’ll see this:

```
R2#
%HA_EM-6-LOG: TIMER : DEBUG(cli_lib) : : CTL : cli_open called.
%HA_EM-6-LOG: TIMER : DEBUG(cli_lib) : : OUT : R2>
%HA_EM-6-LOG: TIMER : DEBUG(cli_lib) : : IN  : R2>enable
%HA_EM-6-LOG: TIMER : DEBUG(cli_lib) : : OUT : R2#
%HA_EM-6-LOG: TIMER : DEBUG(cli_lib) : : IN  : R2#write memory
%HA_EM-6-LOG: TIMER : DEBUG(cli_lib) : : OUT : Building configuration...
%HA_EM-6-LOG: TIMER : DEBUG(cli_lib) : : OUT : [OK]
%HA_EM-6-LOG: TIMER : DEBUG(cli_lib) : : OUT : R2#
%HA_EM-6-LOG: TIMER: Configuration has been saved
%HA_EM-6-LOG: TIMER : DEBUG(cli_lib) : : CTL : cli_close called.
```

## Other Events and Actions

You have seen a couple of events and actions but EEM has a lot of options. Here’s a list to give you some ideas:

<pre><code><strong>R2(config-applet)#event ?
</strong>  application        Application specific event
  cli                CLI event
  config             Configuration policy event
  counter            Counter event
  env                Environmental event
  interface          Interface event
  ioswdsysmon        IOS WDSysMon event
  ipsla              IPSLA Event
  nf                 NF Event
  none               Manually run policy event
  oir                OIR event
  resource           Resource event
  rf                 Redundancy Facility event
  routing            Routing event
  rpc                Remote Procedure Call event
  snmp               SNMP event
  snmp-notification  SNMP Notification Event
  syslog             Syslog event
  tag                event tag identifier
  timer              Timer event
  track              Tracking object event
</code></pre>

Some other useful events are changes in the routing table, IP SLA, object tracking and configuration changes. There is also a big list of possible actions:

<pre><code><strong>R2(config-applet)#action 1.0 ?
</strong>  add               Add
  append            Append to a variable
  break             Break out of a conditional loop
  cli               Execute a CLI command
  cns-event         Send a CNS event
  comment           add comment
  context           Save or retrieve context information
  continue          Continue to next loop iteration
  counter           Modify a counter value
  decrement         Decrement a variable
  divide            Divide
  else              else conditional
  elseif            elseif conditional
  end               end conditional block
  exit              Exit from applet run
  force-switchover  Force a software switchover
  foreach           foreach loop
  gets              get line of input from active tty
  handle-error      On error action
  help              Read/Set parser help buffer
  if                if conditional
  increment         Increment a variable
  info              Obtain system specific information
  mail              Send an e-mail
  multiply          Multiply
  policy            Run a pre-registered policy
  publish-event     Publish an application specific event
  puts              print data to active tty
  regexp            regular expression match
  reload            Reload system
  set               Set a variable
  snmp-trap         Send an SNMP trap
  string            string commands
  subtract          Subtract
  syslog            Log a syslog message
  track             Read/Set a tracking object
  wait              Wait for a specified amount of time
  while             while loop
</code></pre>

Running CLI commands and sending e-mails are maybe the most important ones but you can also generate SNMP traps or reload the router.

Anyway, that’s the end of this lesson. If you enjoyed this, please share it with your friends and colleagues. If you have any other good EEM examples please leave a comment and I’ll add them here.
