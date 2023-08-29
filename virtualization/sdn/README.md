# SDN

## Introduction to SDN

SDN (Software Defined Networking) is the latest buzzword in IT, getting more popular every year. Some network engineers read or heard stories about how the entire network will be programmed in the future, and now fear that their jobs will be replaced by programmers who know C/C++, Java, or Python inside out.

What is SDN, why is it becoming popular? Are those fears justified?

To answer these questions, we will have to take a closer look at “traditional” networking first. We will discuss the current “limitations” of traditional networking, I will explain what SDN is and how SDN is supposed to solve the “problems” that traditional networking has.

## Traditional Networking

Networking has always been very _traditional_. We have specific network devices like routers, switches, and firewalls that are used for specific tasks.

These network devices are sold by networking vendors like Cisco and often use proprietary hardware. Most of these devices are primarily configured through the CLI, although there are some GUI products like CCP (Cisco Configuration Protocol) for the routers or ASDM for the Cisco ASA firewalls.

A network device, for example, a router has different functions that it has to perform. Think for a moment about some of the things that a router has to do in order to forward an IP packet:

* It has to check the destination IP address in the routing table in order to figure out where to forward the IP packet to.
* Routing protocols like OSPF, EIGRP or BGP are required to learn networks that are installed in the routing table.
* It has to use ARP to figure out the destination MAC address of the next hop or destination and change the destination MAC address in the Ethernet frame.
* The TTL (Time to Live) in the IP packet has to be decreased by 1 and the IP header checksum has to be recalculated.
* The Ethernet frame checksum has to be recalculated.

All these different tasks are separated by different **planes**. There are three planes:

* **control plane**
* **data plane**
* **management plane**

Let’s take a look at the difference between these three planes…

### Control Plane

The control plane is responsible for exchanging routing information, building the ARP table, etc. Here are some tasks that are performed by the control plane:

* Learning MAC addresses to build a switch MAC address table.
* Running STP to create a loop-free topology.
* Building ARP tables.
* Running routing protocols like OSPF, EIGRP, and BGP and building the routing table.

### Data Plane

The data plane is responsible for forwarding traffic. It relies on the information that the control plane supplies. Here are some tasks that the data plane takes care of:

* Encapsulate and de-encapsulate packets.
* Adding or removing headers like the 802.1Q header.
* Matching MAC addresses for forwarding.
* Matching IP destinations in the routing table.
* Change source and destination addresses when using NAT.
* Dropping traffic because of access-lists.

The tasks of the data plane have to be performed as fast as possible which is why the forwarding of traffic is performed by specialized hardware like ASICs and TCAM tables.

### Management Plane

The management plane is used for access and management of our network devices. For example, accessing our device through telnet, SSH or the console port.

When discussing SDN, the control and data plane are the most important to keep in mind. Here’s an illustration of the control and data plane to help you visualize the different planes:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2014/10/control-vs-data-plane.png" alt=""><figcaption></figcaption></figure>

Above you can see the control plane where we use routing protocols like OSPF and EIGRP and some static routing. The best routes are installed in the routing table. Another table that the router has to build is the ARP table.

Information from the routing and ARP table is then used to build the forwarding table. When the router receives an IP packet, it will be able to forward it quickly since the forwarding table has already been built.

If you want to learn exactly how the different planes work and how ASICs / TCAMs are used then you might want to read my [CEF (Cisco Express Forwarding) lesson](https://networklessons.com/cisco/ccnp-encor-350-401/cef-cisco-express-forwarding).

## Limitations of traditional networking

Everything I described above is the way we have done things for the last \~30 years so it’s not like there is something “wrong” with traditional networking. However, nowadays there are some business challenges that ask for different solutions.

Let’s look at a scenario.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/vmware-data-center-topology.png" alt=""><figcaption></figcaption></figure>

Above we see the network infrastructure of a company’s data center. At the bottom, we find a VMware ESXi server with a number of virtual machines. This server is connected to some switches in the access and aggregation layers. We also see two ASAs that protect our server and two routers for access to the outside world. On top, there is another router with a host device.

Let’s say this company has a business requirement for a new application that requires four new virtual machines to be installed on the VMware server. For security reasons, each virtual machine should be in a different VLAN. A user that is using H1 behind R3 should be able to access the application that runs on these virtual machines.

Let’s consider some of the things we have to configure on our network to make this happen:

* The VLANs have to be created on all switches.
* We have to configure a root bridge for the new VLANs.
* We have to assign four new subnets, one for each VLAN.
* We need to create new sub-interfaces with IP addresses on the switches.
* We need to configure VRRP or HSRP on the switches for the new VLANs.
* We have to configure the firewalls to permit access to the new applications / subnets.
* We have to advertise the new subnets in a routing protocol on our switches, routers, and firewalls.

Although there are network automation tools to help us, we often use the CLI to configure all of these devices, one-by-one. It’s a **slow, manual process** that a human has to do. While it only takes a few minutes to spin up a new virtual machine, it might take a few hours for the network team to prepare the network. Changes like these are also typically done during a maintenance window, not during business hours.

Server virtualization is one of the reasons why businesses are looking for something that speeds up the process described above. Before virtualization, we used to have one physical server with a single operating system. Nowadays we have multiple physical servers with hundreds of virtual machines.

These virtual machines are able to move automatically from one physical server to another. When they cross an L3 boundary, you don’t want to wait for the network team to make the required changes to routing or access-lists. It should be automatic.

The “trend” nowadays is that everything should be virtual. It’s not strange to see that this is also happening to networking. Large companies like Cisco that used to sell only proprietary hardware are now also offering virtual routers, ASAs, wireless LAN controllers, etc. that you can run on VMWare servers.

## SDN (Software Defined Networking)

Like the buzzword “cloud” a few years ago, every organization or vendor has a different opinion about what SDN exactly is and different products that they offer.

Traditional networking uses a **distributed model** for the control plane. Protocols like ARP, STP, OSPF, EIGRP, BGP and other run separately on each network device. These network devices communicate with each other but there is no central device that has an overview or that controls the entire network.

One exception here (for those that are familiar with wireless networking) are the wireless LAN controllers (WLC). When you configure a wireless network, you configure everything on the WLC which controls and configures the access points. We don’t have to configure each access point separately anymore, it’s all done by the WLC.

With SDN, we use a **central controller for the control plane**. Depending on the vendor’s SDN solution, this could mean that the SDN controller takes over the control plane 100% or that it only has insight in the control plane of all network devices in the network. The SDN controller could be a physical hardware device or a virtual machine.

Here’s an illustration to help you visualize this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/sdn-controller-data-plane-switches.png" alt=""><figcaption></figcaption></figure>

Above you can see the SDN controller which is responsible for the control plane. The switches are now just “dumb” devices that **only have a data plane, no control plane**. The SDN controller is responsible for _feeding_ the data plane of these switches with information from its control plane.

There are some advantages and disadvantages of having a distributed vs a central control plane. One of the advantages of having a central controller is that we can configure the entire network from a single device. This controller has full access and insight of everything that is happening in our network.

Let’s add some more detail to this story. The SDN controller uses two special interfaces, take a look at the image below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/sdn-controller-northbound-southbound.png" alt=""><figcaption></figcaption></figure>

The interfaces are called the **northbound interface (NBI)** and **southbound interface (SBI)**. Let me explain both…

### Southbound Interface

The SDN controller has to communicate with our network devices in order to program the data plane. This is done through the southbound interface. This is not a physical interface but a software interface, often an API (Application Programming Interface).

An API is a software interface that allows an application to give access to other applications by using pre-defined functions and data structures. I’ll explain more about this in a minute.

Some popular southbound interfaces are:

* OpenFlow: this is probably the most popular SBI at the moment, it’s an open source protocol from the [Open Networking Foundation](https://www.opennetworking.org/). There are quite a few network devices and SDN controllers that support OpenFlow.
* Cisco OpFlex: this is Cisco’s answer to OpenFlow. It’s also an open source protocol which has been submitted to the IETF for standardization.
* CLI: Cisco offers APIC-EM which is an SDN solution for the current generation of routers and switches. It uses protocols that are available on current generation hardware like telnet, SSH, and SNMP.

### Northbound Interface

The northbound interface is used to access the SDN controller itself. This allows a network administrator to access the SDN to configure it or to retrieve information from it. This could be done through a GUI but it also offers an API which allows other applications access to the SDN controller. You can use this to write scripts and automate your network administration. Here are some examples:

* List information from all network devices in your network.
* Show the status of all physical interfaces in the network.
* Add a new VLAN on all your switches.
* Show the topology of your entire network.
* Automatically configure IP addresses, routing, and access-lists when a new virtual machine is created.

Here’s an illustration to help you visualize this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/sdn-northbound-interface-api-options.png" alt=""><figcaption></figcaption></figure>

Through the API, multiple applications are able to access the SDN controller:

* A user that is using a GUI to retrieve information about the network from the SDN controller. Behind the scenes, the GUI is using the API.
* Scripts that are written in Java or Python can use the API to retrieve information from the SDN controller or configure the network.
* Other applications are able to access the SDN controller. Perhaps an application that automatically configures the network once a new virtual machine is created on a VMware ESXi server.

### REST API

I have mentioned a few times that the north- and southbound interfaces use APIs. Let’s take a closer look at what an API is. SDN controllers typically use a **REST API (Representational State Transfer)**.

The REST API uses HTTP messages to send and receive information between the SDN controller and another application. It uses the same HTTP messages that you use when you browse a webpage on the Internet or when you enter a contact form online:

* HTTP GET: used when we want to retrieve information.
* HTTP POST/PUT: used when we want to upload or update information.

It is similar browsing a webpage, only this time, you are not requesting a webpage or picture but a particular object from the SDN controller, for example, a list with all VLANs in the network.

When the SDN controller receives the HTTP GET request, it will reply with an HTTP GET response with the information that was requested. This information is delivered in a common data format. The two most used data formats are:

* JSON (JavaScript Object Notation)
* XML (eXtensible Markup Language)

Here’s an example to help you visualize this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/sdn-controller-northbound-http-get.png" alt=""><figcaption></figcaption></figure>

Above we have a python script that is using HTTP GET to fetch the following URL through the API:

https://192.168.1.1:8443/sdn/v2.0/net/nodes

This URL will retrieve some of the variables that are available, for example, information about all nodes (hosts) on the network.

Once the API receives this, it will respond with an HTTP GET response message:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2016/09/sdn-controller-northbound-http-get-response.png" alt=""><figcaption></figcaption></figure>

The variables that were requested will be supplied in JSON format. Here’s what this looks like:

```
{
	"nodes": [
		{
		"ip": "172.16.1.1",
		"mac": "fa16.3e5d.f1f4",
		"vid": 0,
		"dpid": "00:00:00:00:00:00:00:03",
		"port": 1
	}, {
		"ip": "172.16.1.2",
		"mac": "fa16.3e5d.f1f5",
		"vid": 0,
		"dpid": "00:00:00:00:00:00:00:03",
		"port": 2
	}
]
}
```

Even if you have never seen JSON before, the output above is easy to read. It tells us that we have two nodes on the network, their IP, and MAC addresses.

## Conclusion

I hope this lesson has helped to get a global understanding of what SDN is about and why the market is looking for solutions like this. In some other lessons, I will give you some configuration examples of SDN solutions like Open SDN, Openflow, [OpenDayLight](https://networklessons.com/cisco/ccnp-encor-350-401/introduction-to-sdn-opendaylight), Cisco ACI and [Cisco APIC-EM](https://networklessons.com/tag/sdn/introduction-to-apic-em).

Time will tell what networking will look like in the future. It’s likely that we will be using APIs more often than the CLI in the future. Is this a problem? If you ask me, not really. First of all, a programmer might be very strong in programming languages like C/C+, Java or Python but he/she also requires networking knowledge. Understanding how to use a programming language to run some basic scripts will be very helpful, you don’t have to build an entire application from scratch…that’s a programmer’s job.

It might be wise though to spend some time learning a programming language like Python and how to use APIs. At the moment we don’t know how popular SDN will be but this scenario might be a bit similar to what happened to analog telephony. There are a lot of “telephony veterans” that refused to learn about VoIP because they thought analog telephony has always worked. A few years later, they are dinosaurs in a world dominated by VoIP…
