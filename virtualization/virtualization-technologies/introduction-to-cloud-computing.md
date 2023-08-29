# Introduction to Cloud Computing

Cloud computing is an approach where **everything is delivered as a service** by cloud providers or the IT department of your company. Cloud is used a lot as a marketing buzzword, in this lesson, I’ll explain what cloud computing is about. To do so, we have to start with a short story about physical servers, server virtualization and what a typical IT workflow looks like.

## Bare Metal Servers

Before server virtualization, you would only see **bare metal servers**. A bare metal server is a physical server that runs a **single operating system** like Microsoft Windows Server or Linux. This is from a time where CPUs only had 1-2 cores and not a lot a RAM.

Servers are similar to your computer at home, the main difference is that they are built to run 24/7 and usually have more reliable hardware. Some have redundant power supplies, hard disks, etc.

They come in different forms and shapes, the tower servers are often used in office buildings:

The bare metal server has a single operating system and can be used for one or multiple applications. For example, for small businesses, Microsoft Small Business Server has always been a popular office solution. It runs on 1 or 2 physical servers and offers:

* Directory service
* DNS server
* E-mail server
* Web server
* File server
* And some other services

In enterprise environments, it’s more common to see that a physical server is used for each “role”. One web server, one DNS server, etc.

In data centers, physical space is expensive so it’s more common to see rack servers there:

Rack servers are placed in server cabinets:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/data-center-room.jpg" alt="The Server Control Center For An Internet Provider Is Called A Data Center. Generative Ai" height="737" width="1106"><figcaption><p>The server control center for an internet provider is called a data center. Generative AI</p></figcaption></figure>

Here’s an example of an HP Proliant rack server in a rack cabinet:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/proliant-dl380-g5.jpeg" alt=""><figcaption></figcaption></figure>

Later, blade servers were introduced that offer even more computing power / memory and use less space:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/cisco-ucs-blade-server.jpg" alt=""><figcaption></figcaption></figure>

Above you see a blade enclosure that fits in a server rack. It offers networking, power, and cooling. The _blade servers_ have CPUs, memory, and storage. They fit in the slots of the blade enclosure.

When you buy a physical server, you have to think of the required resources. How much memory does the server need? How much disk space? How fast should the CPUs be? etc. You also have to take future growth into account.

## Server Virtualization

In the last decade, the number of cores in a single processor has grown rapidly. We went from single core processors to dual core processors, quad core processors and now there are even processors with more than 10-20 cores. There is a also a technique called _hyperthreading_ where we have two virtual cores for each physical core. This allows the operating system to execute two threads at the same time. Here is an example of an Intel I7 processor:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/intel-i7-4790k-8-cores.png" alt=""><figcaption></figcaption></figure>

Above you can see that this CPU has 4 physical cores and 8 virtual (logical) cores.

The amount of (affordable) RAM has also increased a lot. A single physical server often has way more CPU and RAM resources than what is required for a single operating system. Nowadays, we use a lot of server virtualization which means that we run **multiple virtual machines on a single physical server**.

All hardware of the VM (virtual machine) is virtualized, its CPU(s), RAM, hard disks, network card, etc. For each virtual machine you create, you can decide how many CPU(s), RAM, etc. it will have.

The server virtualization software that we run on the physical server (called the host when talking about server virtualization) is called the **hypervisor**. The hypervisor is what manages the VMS and where you configure how much RAM/CPUs/disk space you assign to each VM.

Here’s an image of what a bare metal server looks like that runs a single operating system:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/physical-server-concept.png" alt=""><figcaption></figcaption></figure>

Above you can see we have the hardware that runs an operating system and an application. Here’s what it looks like when we go virtual:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/server-virtualization-concept.png" alt=""><figcaption></figcaption></figure>

On top of the hardware, we run the hypervisor. On top of the hypervisor, we run multiple virtual machines with an operating system each.

Here are some companies/products that offer server virtualization solutions:

* VMware
* Microsoft HyperV
* Citrix Xenserver
* Red Hat KVM

If your hardware crashes then suddenly all your virtual machines will be gone too. Besides the hypervisor, these companies also offer products so that virtual machines can be automatically created and moved from one hypervisor to another.

### Virtual Networking

Physical servers have one or more NICs (Network interface card) that are connected to a network switch. What about virtual servers? Virtual machines have virtualized hardware and that includes their NICs, which we call vNIC (Virtual NICs). Somehow we have to connect these vNICs to our network, this is done with a virtual switch. Take a look at this example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/hypervisor-virtual-switch-concept.png" alt=""><figcaption></figcaption></figure>

Above you can see that the virtual machines each have a vNIC that is connected to a virtual switch. The virtual switch is connected to a physical switch through the physical NIC of the server that the hypervisor runs on.

In the picture above there is only one physical NIC. In production networks, we usually use two or more physical NICs for redundancy and to make sure there is enough bandwidth for all virtual NICs.

The virtual switch is supplied by the hypervisor vendor or you can use an external virtual switch product like the Cisco Nexus 1000v switch. This allows you to use the same switch features on your virtual switch as you use on your physical switches.

### Physical Data Center Network

We just talked about how virtual machines are connected to a physical switch through a virtual switch. Let’s zoom out a bit…what does it look like in a real data center network where we have racks full of servers? There are two common options, let’s take a look at both.

#### **TOR (Top of Rack)**

The top of rack designs has network switches at the top of each server cabinet / rack. The servers are located below the ToR switches and for redundancy reasons, connected to both ToR switches:

The ToR switches are connected to distribution layer switches. One of the advantages of this setup is that most of your cabling remains within the rack, the only network cables that leave the rack are those from the ToR switches to the distribution layer switches. One of the disadvantages is that you need quite some ToR switches and depending on how much servers you have in your rack, not all switch ports on the ToR switches will be used.

#### **EOR (End of Row)**

With the end of row design, there are no switches in the racks and all servers are directly connected to EoR (End of Row) switches that are located in a separate rack:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/end-of-rack-architecture.png" alt=""><figcaption></figcaption></figure>

Some of the advantages of this setup is that you don’t need as many switches, there are less unused switch ports so overall port utilization is better. One of the disadvantages is that you need a lot of cabling from your server racks to the racks where the EoR switches are located.

### Workflow and Service

We talked about physical servers, virtualization and a bit of how these virtual / physical servers are connected in a data center. One of the things that hasn’t changed much throughout the years is the **workflow**. Let me give you an example.

Let’s say we have an enterprise company with thousands of users. They have their own data center with racks full of servers and hundreds of virtual machines.

This company has different departments, including one with developers that work on a new web application. One of the developers wants to test his/her applications and requires a web server. The workflow typically looks like this:

1. The developers sends a request to the IP department and explains he/she has a new application that has to run on a new web server. The application requires the Apache web server software and a MySQL database.
2. The IT department looks at the requirements and creates a new virtual machine on the hypervisor on one of the servers. The new virtual machine is installed with an operating system and all required software.
3. The IT department reports back to the developer who will then test the new application on the new server.

The workflow described above is how it’s been for a long time and it works. The “problem” however, is that it quite a slow process.

There is always a human in between (the IT department) that has to do some work to offer the “service” to the customer (the developer) is looking for.

## Cloud Computing

The idea behind cloud computing is that we offer different services, a customer should be able to request the service and receive the service right away. There is **no human in between** that has to look at the customer’s request, process it and report back. Everything is automated.

That’s not the only advantage of cloud computing, though. NIST (National Institute of Standards and Technology has a good definition of cloud computing:

* **On-demand** **self-service**: a customer should be able to get a service automatically, without having to wait for a human to provide it for her. The customer should also be able to terminate the service by herself.
* **Broad network access**: the service should be available using a variety of different platforms including computers, tablets, smartphones. We should also be able to reach the service using different connections, including the Internet or private WAN connections.
* **Resource pooling**: the cloud provider should not assign fixed resources to the service but it has to be dynamic. For example, when a website suddenly receives a lot of traffic, multiple web servers should be created automatically so that we can deal with the incoming traffic.
* **Rapid elasticity**:  to the customer, the resources should look to be unlimited. For example, there are some cloud backup providers that you can use for remote backups. You pay for the service and behind the scenes, they will take care that there is enough storage space for you to upload your files to. It doesn’t matter if you upload 1 GB or 100 TB.
* **Measured service**: the cloud provider measures all resource usage for billing and transparency.

### Services model

Instead of buying a product, we pay for a service. When we talk about cloud computing, we use the “as a service” terminology a lot. These three are the most common:

* (IaaS) Infrastructure as a service
* (PaaS) Platform as a service
* (SaaS) Software as a service

Let me walk you through them and give you some examples of each.

#### **IaaS (Infrastructure as a service)**

**Iaas offers network infrastructure as a service.** These are network devices like virtual machines, routers, switches, firewalls, load balancers, and storage.

Let me give you an example of how Amazon AWS offers virtual machines to its customers. When you create a new virtual machine, you can choose which operating system you want to use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/amazon-aws-choose-amazon-machine-image-ami.png" alt=""><figcaption></figcaption></figure>

Once you have decided which operating system to use, you can choose an instance type with a certain number of CPU cores, an X amount of memory, storage and network performance:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/amazon-aws-choose-instance-type.png" alt=""><figcaption></figcaption></figure>

Within a few minutes, your new virtual machine will be up and running. Don’t need it anymore? It’s billed per hour and you can delete it whenever you want.

#### **PaaS (Platform as a Service)**

Let’s say you are a developer who is working on a new web application. Your application has some server requirements.

As a developer, you care about your writing your application and you really don’t want to be bothered with the details of virtual machines, installing software, etc.

Instead, you can use a PaaS solution like Amazon AWS Beanstalk. This is a virtual machine with a preinstalled OS and all software required to run your application:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/amazon-aws-beanstalk.png" alt=""><figcaption></figcaption></figure>

The only thing you have to do is upload your application and run it. Another example is Google’s App Engine:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/google-app-engine.png" alt=""><figcaption></figcaption></figure>

Google App Engine lets you develop and host apps, they take care of the underlying infrastructure for you. The only thing you have to do is create your app and upload it.

#### **SaaS (Software as a Service)**

_Saas (Software as a Service)_ means the customer signs up for an application and can use it right away, without having to install anything. Even though this term might be new, we have all been using this for years. Some examples:

* Freshbooks: sign up and you can use the accounting software right away from any location.
* Gmail: sign up and you can send and receive e-mail right away.
* Microsoft Office 365: sign up and you can use the web versions of word, excel, etc. right away.
* Microsoft OneDrive: sign up and you have a shared network drive that is reachable from any location and on multiple devices.

With SaaS, you pay for the application and the cloud provider takes care of installation, maintaining the virtual servers etc.

### Public Cloud

The services I showed you above are all offered by a cloud provider, we call this the **public cloud**_._ The largest cloud providers are:

* Amazon AWS
* Microsoft Azure
* IBM
* Google Cloud Platform

Amazon AWS is by far the largest cloud provider at the moment, Here’s an overview with the services they offer, it’s quite a big list:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/amazon-aws-services.png" alt=""><figcaption></figcaption></figure>

It’s no coincidence that most of my screenshots are from Amazon AWS. It is the largest cloud provider by far and networklessons.com also runs on Amazon AWS.

### Private Cloud

Remember the story about workflow where the IT department manually creates new virtual machines and installs new software when requested? With a private cloud, a company changes their workflow so that they operate in a similar was as the public cloud providers.

The IT department will create a service catalog that lists all services that they can offer to their users. Behind the scenes, everything is automated so that when a user requests a service, it is provisioned automatically.

Next time one of the company users need something. He or she can click on a few buttons and a new virtual machine with the required software is launched.

## WAN Traffic Path to Cloud Services

You now have an idea what the cloud is about, let’s take a closer look at some of the different options of how we can connect to the cloud.

### Internet

Using the Internet to connect to the cloud is a common option. There are some advantages and disadvantages, however. Let’s look at an example of an enterprise network that uses the services of a cloud provider:

Above we see that all virtual machines are located at the cloud provider. The enterprise network doesn’t have any servers or virtual machines anymore, only users that require access to the applications that run on the virtual machines.

The advantages of using the Internet as your WAN connection to the cloud are:

* **Cost**: Internet access is cheap compared to private WAN options.
* **Availability**: it’s easy to get an Internet connection and it’s available almost everywhere.
* **Migration**: want to switch from one cloud provider to another? All cloud providers are connected to the Internet so you don’t have to switch connections.
* **Mobile users**: if you have a lot of mobile users then they will be able to access your applications whenever they have an Internet connection.

Some of the disadvantages:

* **Security**: the Internet is a public network so it’s not a very safe place. Attackers might attempt man-in-the-middle attacks to snoop on the traffic between your users and the cloud.
* **Bandwidth**: depending on the type of applications and the number of users, your Internet connection might not have enough bandwidth for all users to access their applications.
* **QoS (Quality of Service)**: the Internet is best-effort only, there is no quality of service. If you have any applications that are sensitive to delay and/or packet loss then you might run into issues.
* **SLA:** most Internet providers don’t offer any SLAs (Service Level Agreements) that guarantee a certain bandwidth or availability. If you outsource all or most of your applications to the cloud, you will be very dependent on your Internet connection.

Most cloud providers also support IPsec VPN, allowing you to create a site-to-site VPN tunnel between your Enterprise network and the Cloud provider.

### Private WAN

An alternative connection method is the private WAN. This is a dedicated connection from your site to the cloud provider.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/private-wan-enterprise-cloud-provider.png" alt=""><figcaption></figcaption></figure>

Advantages:

* **Bandwidth**: private WAN connections offer a higher bandwidth than most Internet connections.
* **SLA**: does offer service level agreements that guarantee a certain bandwidth and availability.

Disadvantages:

* **Cost**: private WAN connections cost more than regular Internet connections.
* **Availability**: takes time to install the new connection.
* **Flexibility**: you are stuck to one cloud provider.

Here are some examples of private WAN connections:

* Microsoft Azure ExpressRoute
* Amazon AWS Direct Connect

### Intercloud Exchange

What if you want to use multiple cloud providers and the advantages of private WAN connections? What if you want to migrate from one cloud provider to another? You could get multiple private WAN connections but there is a better option called **intercloud exchanges**.  These are providers that are connected to multiple cloud providers and offer you a private WAN. Take a look at the picture below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/enterprise-intercloud-exchange-private-wans.png" alt=""><figcaption></figcaption></figure>

The intercloud exchange can offer you a connection to one or more cloud providers without having to switch your private WAN connection. You get the advantage of a private WAN, without being stuck to one cloud provider.

### Virtual Network Functions

Most cloud providers offer some basic networking functions. You can choose if your virtual machines have public IP addresses so that you can directly reach them from the Internet or if they use private IP addresses with a router and/or load balancer in front of them. The router and the load balancer that the cloud provider offers, however, have limited options.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/cloud-provider-router-slb.png" alt=""><figcaption></figcaption></figure>

What if you require some specific features on your router? Perhaps you use DMVPN in your network and you also need this at the cloud provider. Perhaps you have some specific firewall features that you would like to use. The cloud provider might not offer this directly, but it is possible to use **VNFs (Virtual Network Function)**. A VNF is the virtual version of your favorite router, firewall or other network devices. Here are two examples:

* Cisco Cloud Services Router 1000V Series: this CSR is a virtual IOS XE router that offers the same features as physical Cisco IOS XE routers.
* Cisco ASAv: this is the virtual version of the Cisco ASA firewall.

You can add them to your cloud network if needed:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/cloud-virtual-network-function.png" alt=""><figcaption></figcaption></figure>

This will give you the same networking features at the cloud provider network as those that you use on your own enterprise network.

## Conclusion

You have now learned some of the basics of cloud computing:

* How we started with physical servers and how we moved to server virtualization.
* That we use hypervisors to run virtual machines on a single physical server.
* How virtual machines use virtual NICs and virtual switches to connect to the rest of the network.
* What a physical data center looks like with ToR (Top of Rack) and EoR (End of Row) designs.
* That the classic workflow involved an IT department that manually created virtual machines and installed applications.
* How cloud computing is about self-service, where we don’t require a human in between to install virtual machines and/or applications.
* The difference between IaaS, SaaS and PaaS.
* The different WAN options to connect to the cloud.
* What virtual network functions are.
