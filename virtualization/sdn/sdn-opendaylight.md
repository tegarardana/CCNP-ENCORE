# SDN OpenDayLight

OpenDaylight is an open source SDN controller / framework, hosted by the Linux Foundation. It’s one of the more popular (open source) SDN controllers at the moment.

One of the southbound interface protocols it supports is OpenFlow. To test OpenDaylight, we’ll need some switches that support OpenFlow.

You could buy some hardware that supports OpenFlow but a great alternative is [Mininet](http://mininet.org/).

Mininet allows you to run a virtual network on your own computer with devices that support OpenFlow.

In this lesson, I’ll show you how to run a virtual network with two OpenFlow switches that are controlled by our OpenDaylight SDN controller. You will also see some examples of how you can use the RESTCONF API to interact with your controller.

Here’s the virtual network that we are going to build:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/10/opendaylight-sample-topology.png" alt=""><figcaption></figcaption></figure>

The above network has an OpenDaylight SDN controller on top and two OpenFlow switches that are controlled by the SDN controller. Two hosts will be connected to our switches.

To achieve this, we will use two virtual machines:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/10/mininet-opendaylight-topology.png" alt=""><figcaption></figcaption></figure>

Above you see two virtual machines. On the left side, we have our Mininet server with two interfaces. The eth0 interface is using DHCP client, I only use this interface to SSH into the box. The eth1 interface will be used by the virtual switches communicate with the OpenDaylight controller.

Our OpenDaylight controller also has two interfaces. The ens160 interface will be used to SSH into the box and to access the GUI / API. The ens192 interface is used to communicate with the virtual switches.

One advantage of having Mininet in a separate virtual machine is that you can easily connect your virtual network to different remote SDN controllers.

## OpenDaylight Installation

To create the OpenDaylight virtual machine I downloaded the latest [Ubuntu Server 16.04.1 LTS image](http://www.ubuntu.com/download/server). I created a new virtual machine in VMWare workstation that has two network interfaces.

OpenDaylight is actively developed so it’s possible that some of my installation instructions are different for newer releases. I used a Ubuntu 16.04.1 LTS server image for this setup and the latest “Boron” release from OpenDaylight. You can also check the OpenDaylight wiki for installation instructions.

Once the new virtual machine has booted, we can start with the installation.

If you don’t want to build this VM yourself, feel free to [download my VM here](https://s3.amazonaws.com/nwlpublic/OpenDayLightBoron.ova). Username and password are both “vmware”.

### Network Interface

By default, only the first network interface will be active and it is configured to use DHCP client. We will have to add the second network interface ourselves. The most recent versions of Ubuntu use a different naming scheme for interfaces. To figure out the names of your interfaces you can use the following command:

<pre><code><strong>$ ip link
</strong>1: lo: &#x3C;LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens160: &#x3C;BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:d2:92:c6 brd ff:ff:ff:ff:ff:ff
3: ens192: &#x3C;BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:d2:92:d0 brd ff:ff:ff:ff:ff:ff
</code></pre>

This tells me that my virtual machine has an ens160 and 192 interface. Let’s edit the network configuration to activate the second interface:

<pre><code><strong>$ sudo vim /etc/network/interfaces
</strong></code></pre>

Add the following entry:

```
# Second interface
auto ens192
iface ens192 inet static
address 192.168.1.254
netmask 255.255.255.0
```

To activate these changes, you can reboot your virtual machine or use the **sudo ifup ens192** command.

### Java

Now we can continue with our installation. OpenDaylight is a Java program so we need to install the runtime:

<pre><code><strong>$ sudo apt-get install default-jre-headless
</strong></code></pre>

Open the following file in your favorite text editor:

<pre><code><strong>$ vim ~/.bashrc
</strong></code></pre>

And add the following line to set the JAVA\_HOME variable:

```
export JAVA_HOME=/usr/lib/jvm/default-java
```

Save the file and execute the bashrc file:

<pre><code><strong>$ source ~/.bashrc
</strong></code></pre>

### OpenDayLight

Let’s download OpenDaylight. You can download the [latest version here](https://www.opendaylight.org/downloads).

<pre><code><strong>$ wget https://nexus.opendaylight.org/content/groups/public/org/opendaylight/integration/distribution-karaf/0.5.0-Boron/distribution-karaf-0.5.0-Boron.zip
</strong></code></pre>

To unzip the file we have to install unzip:

<pre><code><strong>$ sudo apt-get install unzip
</strong></code></pre>

Let’s unzip the file:

<pre><code><strong>$ unzip distribution-karaf-0.5.0-Boron.zip
</strong></code></pre>

We can now start the controller:

<pre><code><strong>$ cd distribution-karaf-0.5.0-Boron/
</strong><strong>$ ./bin/karaf
</strong></code></pre>

OpenDaylight uses a karaf container. This is technology from Apache that allows you to run everything from a single folder. The single ZIP file that we downloaded contains everything, it has the OpenDaylight software but also includes the Apache webserver and some other things. There’s no need to install a webserver yourself.

It will take a minute to start. Once it is started, you will see the following screen:

```
Apache Karaf starting up. Press Enter to open the shell now...
100% [========================================================================]

Karaf started in 69s. Bundle stats: 314 active, 314 total
                                                                                           
    ________                       ________                .__  .__       .__     __       
    \_____  \ ______   ____   ____ \______ \ _____  ___.__.|  | |__| ____ |  |___/  |_     
     /   |   \\____ \_/ __ \ /    \ |    |  \\__  \<   |  ||  | |  |/ ___\|  |  \   __\    
    /    |    \  |_> >  ___/|   |  \|    `   \/ __ \\___  ||  |_|  / /_/  >   Y  \  |      
    \_______  /   __/ \___  >___|  /_______  (____  / ____||____/__\___  /|___|  /__|      
            \/|__|        \/     \/        \/     \/\/            /_____/      \/          
                                                                                           

Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown OpenDaylight.

opendaylight-user@root>
```

The controller comes with some basic features but if we want to do anything useful, we have to add some additional features. In this lesson, I want to use the GUI, the RESTCONF API and some basic L2 switching functionality to learn MAC addresses. We’ll have to install the following features:

<pre><code><strong>opendaylight-user@root>feature:install odl-restconf odl-l2switch-switch odl-mdsal-apidocs odl-dlux-all
</strong></code></pre>

We can now try to access the controller from a browser. Enter the following URL:

http://ip-address-of-opendaylight-:8181/index.html#login

You will see the following login screen:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/opendaylight-login.png" alt=""><figcaption></figcaption></figure>

The default username and password are “admin”. Click on login and you will see the topology screen:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/opendaylight-controls-empty.png" alt=""><figcaption></figcaption></figure>

We are able to log in but the topology picture is empty. This makes sense since we don’t have any switches at the moment. Time to dive into Mininet.

## Mininet

Mininet allows us to run a virtual network with routers, switches, and hosts. You could install this yourself on a Linux server but you can also download a[ pre-packaged virtual machine](http://mininet.org/download/) from the Mininet website that has everything you need.

You can also [download my VM](https://s3.amazonaws.com/nwlpublic/Mininet221.ova) which has the second network interface with static IP address. Username and password are both “mininet”.

I access the Mininet virtual machine through SSH and changed the network interface so that it has the correct IP address. Let’s try if we are able to reach the OpenDaylight controller from our Mininet VM:

<pre><code><strong>$ ping 192.168.1.254 -c 3
</strong>PING 192.168.1.254 (192.168.1.254) 56(84) bytes of data.
64 bytes from 192.168.1.254: icmp_seq=1 ttl=64 time=0.248 ms
64 bytes from 192.168.1.254: icmp_seq=2 ttl=64 time=0.284 ms
64 bytes from 192.168.1.254: icmp_seq=3 ttl=64 time=0.220 ms

--- 192.168.1.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.220/0.250/0.284/0.031 ms
</code></pre>

Our MiniNet VM is able to reach OpenDaylight.

We will start with something simple. There are a lot of fancy topologies we can build but for now, let’s build a network that:

* has two virtual switches that support Openflow.
* has two virtual hosts.
* uses OpenDaylight as the remote SDN controller.

You can use the following command to launch this:

<pre><code><strong>$ sudo mn --topo linear,2 --mac --controller=remote,ip=192.168.1.254,port=6633 --switch ovs,protocols=OpenFlow10
</strong>*** Creating network
*** Adding controller
*** Adding hosts:
h1 h2 
*** Adding switches:
s1 s2 
*** Adding links:
(h1, s1) (h2, s2) (s2, s1) 
*** Configuring hosts
h1 h2 
*** Starting controller
c0 
*** Starting 2 switches
s1 s2 ...
*** Starting CLI:
mininet>
</code></pre>

Our virtual network is now up and running.

## OpenDaylight GUI

Let’s take another look at the OpenDaylight GUI. If you refresh the topology page, you should see something:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/opendaylight-topology-two-openflow-switches.png" alt=""><figcaption></figcaption></figure>

Right now we only see two switches. Our virtual network also has two hosts but our controller hasn’t seen these before.

Mininet allows us to generate traffic. A quick way to do this is by using the **pingall** command. As the name implies, it sends some pings between all devices:

<pre><code><strong>mininet> pingall
</strong>*** Ping: testing ping reachability
h1 -> h2 
h2 -> h1 
*** Results: 0% dropped (2/2 received)
</code></pre>

Our pings from host 1 to host 2 and vice versa are working. If you refresh your web browser, you should be able to see something else on the topology page:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/opendaylight-topology-two-openflow-switches-and-hosts.png" alt=""><figcaption></figcaption></figure>

Are hosts are now showing up in the topology.

## RESTCONF API

The GUI of OpenDaylight is nice but also very limited. If we want to monitor and/or configure our SDN controller, we have to use the RESTCONF API.

There are a number of different methods to use the API, I’ll show you some different options. First, let me explain what RESTCONF is in a nutshell.

For the last couple of decades, we have been using the CLI to configure our network equipment. A lot of network automation tools also use CLI scripting.

The CLI however, has been created for humans. We use configuration commands to configure everything and show/debug commands to verify our work. For us, this works very well but for software, not so much.It can be a pain to write scripts that enter commands and wait for a result or check the output of show commands. The syntax and output of commands can also change in different software versions which means you have to update your scripts.

An alternative to the CLI is SNMP. SNMP can be used for configuration but in reality, it’s mostly only used for monitoring of the network. One of the issues with SNMP is that it can be hard to figure out what MIBs to use.

To make network automation easier, the IETF developed **NETCONF**.

NETCONF is a management protocol that allows us to install, manipulate, and delete configurations of network devices. It uses XML for data encoding and RPC for transport. It can be used as an alternative to the CLI to configure your devices. NETCONF can also be used to fetch statistics from your network devices.

Besides NETCONF, we use a data modeling language for NETCONF called YANG.

YANG uses a hierarchical structure and makes things easy to read for humans.

Last but not least, we have something called RESTCONF. This is an API interface over HTTP that allows us to access data from YANG.

### Yang UI

We can access Yang from the OpenDaylight GUI so let’s take a look:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/opendaylight-yang-ui.png" alt=""><figcaption></figcaption></figure>

Above you see a lot of different objects. Let’s see if we can fetch the current (operational) network topology through Yang:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/opendaylight-yang-ui-network-topology.png" alt=""><figcaption></figcaption></figure>

Once you click on the network topology, Yang will automatically show you the CONFREST API URL that it uses to retrieve this information:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/opendaylight-yang-ui-rest-api-http-get.png" alt=""><figcaption></figcaption></figure>

You can copy this URL and paste it in notepad so we can use it later:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/opendaylight-yang-ui-clipboard.png" alt=""><figcaption></figcaption></figure>

Here’s the URL that I got:

> http://10.56.100.183:8181/restconf/operational/network-topology:network-topology

Click on the Send button:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/opendaylight-yang-ui-send.png" alt=""><figcaption></figcaption></figure>

And you will see the operational network topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/opendaylight-yang-ui-rest-api-http-get-host.png" alt=""><figcaption></figcaption></figure>

Above you can see the information about our current topology, including the MAC and IP addresses of our hosts.

### Postman

The Yang UI is nice to browse around a bit but it’s not very useful if you really want to work with the RESTCONF API.

There’s a great app called Postman that you can [download here](https://www.getpostman.com/).

It allows you to communicate with APIs, save requests, etc. Here’s a screenshot of the main screen:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/postman-main-screen.png" alt=""><figcaption></figcaption></figure>

We found the URL that showed us the topology in the Yang UI, let’s try the same URL in Postman:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/postman-get-url.png" alt=""><figcaption></figcaption></figure>

Before you click on the send button, make sure you enter your credentials. We need to be authorized by the API to do anything. Click on the Authorization button and enter the credentials of OpenDaylight (admin / admin):

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/postman-authorization.png" alt=""><figcaption></figcaption></figure>

Once you hit the Send button, you will get a response in JSON:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/postman-body-json.png" alt=""><figcaption></figcaption></figure>

Above you can see the same output as what we saw in Yang UI but this time, it’s in JSON format. One of the advantages of Postman is that you can save your requests or download collections of requests from others. For example, here’s a [collection of OpenDaylight requests provided by CiscoDevNet](https://github.com/CiscoDevNet/opendaylight-sample-apps/tree/master/postman-collections).

### Python

Postman works great to work with the API but one of the goals of SDN is network automation. Let me show you an example of where we use Python to request the same URL that we used in Yang UI and Postman to fetch the topology information.

Once the script has retrieved the topology information, it will print the MAC and IP addresses of all hosts in the network:

```
import requests

#OpenDayLight RESTCONF API settings.
odl_url = 'http://10.56.100.183:8181/restconf/operational/network-topology:network-topology'
odl_username = 'admin'
odl_password = 'admin'

# Fetch information from API.
response = requests.get(odl_url, auth=(odl_username, odl_password))

# Find information about nodes in retrieved JSON file.
for nodes in response.json()['network-topology']['topology']:

    # Walk through all node information.
    node_info = nodes['node']

    # Look for MAC and IP addresses in node information.
    for node in node_info:
        try:
            ip_address = node['host-tracker-service:addresses'][0]['ip']
            mac_address = node['host-tracker-service:addresses'][0]['mac']
            print('Found host with MAC address %s and IP address %s' % (mac_address, ip_address))
        except:
            pass
```

Let’s run the script above:

<pre><code><strong>$ python topology_information.py
</strong></code></pre>

It will print the following:

```
Found host with MAC address 00:00:00:00:00:02 and IP address 10.0.0.2
Found host with MAC address 00:00:00:00:00:01 and IP address 10.0.0.1
```

It’s a simple script but it does demonstrate how we can automate different network tasks.

## Conclusion

In this lesson, you have learned how to start the OpenDaylight controller and a virtual network with Mininet. We have also seen how you can work with the RESTCONF API using different tools like the Yang UI, Postman and a Python script.

I hope this has been helpful to get a basic understanding of what SDN / OpenDaylight is about. In other lessons, we will look at some more advanced topics like configuring the SDN controller.
