# Introduction to REST API

In this lesson, we’ll take a look at what a REST API is. First of all, what is an API?

We, as network engineers usually use the command-line interface (CLI) or a GUI to configure or monitor our network devices. Parsing show and debug commands with scripts is difficult because these commands are for humans. To interact with applications or network devices, we can use an **Application Programming Interface (API).**

An API is a software interface which allows other applications to communicate with our application.

Roy T. Fielding describes REST in his [dissertation (chapter 5)](https://www.ics.uci.edu/\~fielding/pubs/dissertation/rest\_arch\_style.htm). It’s an acronym for **RE**presentational **S**tate **T**ransfer:

* **Representational** means we transfer the _representation_ of a resource between a server and a client. We use a data format for this representation, typically [JSON or XML](https://networklessons.com/cisco/ccnp-encor-350-401/data-models-and-structures).
* **State Transfer** means that each operation with a REST API is **self-contained**. Each request carries (transfers) all information (state) to complete the operation.

REST APIs **typically use HTTP methods** to retrieve or send information between applications. We use the same HTTP methods when  we use a web browser to visit a website, but now we use them to interact with an application. HTTP has multiple methods, but these four are the most common ones:

* **GET**: A read-only method to retrieve a specified resource.
* **POST**: Submits data to the specified resource to process. The POST method can also create new resources.
* **PUT**: Updates the specified resource by replacing the existing data.
* **DELETE**: Deletes the specified resource.

HTTP is popular so you can use REST APIs in almost any programming language.

I mentioned _resource_ several times but didn’t explain exactly what a resource is. A resource is a “thing” you can access and receive or change its representation. On the web, this could be a document or image. With a REST API, it could be a row in a database.

We access a resource with a **Uniform Resource Locator (URL)**. That’s right, the URLs we also use for websites. A quick example is the following URL:

`https://192.168.1.1:55443/api/v1/interfaces/loopback0`

We can use this URL to access the Loopback 0 resource on a router. I’ll show you in a minute what that looks like in action.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/09/http-get-web-vs-rest-api.png" alt=""><figcaption></figcaption></figure>

An API must meet **6 guiding constraints** if we want to call it a REST API. Another name for an API that meets all constraints is a **RESTful service**. Understanding these constraints in detail is essential for API developers.

The 6 constraints are:

* **Client-server**: The client and server are independent. They interact with each other through requests that the client initiates. The server listens for requests and answers them.
* **Stateless**: The server doesn’t store any state about previous requests. For example, it doesn’t track if a client already requested a resource before. It also doesn’t track which resources were requested by a client.
* **Cacheable**: The server includes a version number in its messages. The client can use this to decide whether it should request a resource again or use the cached data.
* **Uniform interface**: The interface is decoupled from the implementation. There are four sub-constraints:
  * **Identification of resources**: Each resource must be uniquely identifiable via a URI (Uniform Resource Identifier).
  * **Manipulation of resources through representations**: A client can’t interact directly with a server’s resource. For example, you can’t run a SQL query directly from the client against a server’s database. We have to use a representation and neutral data format. When a client wants to update a resource, it has to take the following steps:
    * Request a representation of the resource.
    * Update the representation with new data.
    * Send the representation of the resource to the server.
  * **Self-descriptive messages**: Each message (request or response) must include enough information so the receiver can understand it. A message requires a media type (for example “application/json”) that tells the receiver how to parse the message.
  * **Hypermedia as the engine of application state (HATEOAS)**: You should be able to discover other areas of the API similar to how a user browses a website. A response from the API should include links to other parts of the API. This way, you can figure out how the API works without referring to external documentation.
* **Layered system**: REST allows a layered system. You could deploy the API on one server and the data on another server. You could add additional layers like a load-balancer in between the client and server. The client can’t tell whether it’s communicating with an intermediate or end server. Extra layers should not affect communications between the client and the server.
* **Code on demand (optional)**: This is an optional constraint. The server usually sends a static representation in JSON or XML. The server can optionally include executable code to a client. An example is a Java applet or JavaScript.

REST has no built-in security features, but if needed we can add these. For example, we can use HTTPS for encryption and usernames or tokens for authentication.

## Configuration

Enough theoretical talk for now. Let’s see how we can use a REST API to monitor and configure a Cisco CSR1000v router. Here’s the topology we’ll use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/09/r1-host-python-rest-api-topology.png" alt=""><figcaption></figcaption></figure>

The router has a loopback 0 interface that we’ll access and configure with some Python scripts on my computer.

### Router

Let’s start with the router. This is a CSR1000v router running IOS XE 16.6.1:

<pre><code><strong>R1#show version | include Version
</strong>Cisco IOS XE Software, Version 16.06.01
Cisco IOS Software [Everest], Virtual XE Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 16.6.1, RELEASE SOFTWARE (fc2)
</code></pre>

{% tabs %}
{% tab title="Configuration" %}
Want to take a look for yourself? Here you’ll find the startup configuration of each device.
{% endtab %}

{% tab title="R1" %}
```
hostname R1
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface GigabitEthernet2
 ip address 172.16.1.100 255.255.255.0
 negotiation auto
 no mop enabled
 no mop sysid
!
ip route 0.0.0.0 0.0.0.0 172.16.1.254
!
end
```
{% endtab %}
{% endtabs %}

First, we create a user that has full access to the router:

<pre><code><strong>R1(config)#username admin privilege 15 password admin
</strong></code></pre>

Also, we activate the shared management interface:

<pre><code><strong>R1(config)#virtual-service csr_mgmt
</strong><strong>R1(config-virt-serv)#ip shared host-interface GigabitEthernet 2
</strong><strong>R1(config-virt-serv)#activate
</strong>
% Activating virtual-service 'csr_mgmt', this might take a few minutes. Use 'show virtual-service list' for progress.
</code></pre>

This configuration is all we have to do to enable the REST API. There are some show commands we can use to verify if the REST API is up. The first two commands are to check the status of the virtual service:

<pre><code><strong>R1#show virtual-service list
</strong>Virtual Service List:


Name                    Status             Package Name                         
------------------------------------------------------------------------------
csr_mgmt                Activated          iosxe-remote-mgmt.16.06.01.ova
</code></pre>

If you want more details, try this:

<pre><code><strong>R1#show virtual-service detail name csr_mgmt
</strong>Virtual service csr_mgmt detail
  State                 : Activated
  Owner                 : IOSd
  Package information
    Name                : iosxe-remote-mgmt.16.06.01.ova
    Path                : bootflash:/iosxe-remote-mgmt.16.06.01.ova
    Application
      Name              : csr_mgmt
      Installed version : 2017.6
      Description       : CSR-MGMT
    Signing
      Key type          : Cisco release key
      Method            : SHA-1
    Licensing
      Name              : Not Available
      Version           : Not Available

  Detailed guest status
    
----------------------------------------------------------------------
Process               Status            Uptime           # of restarts
----------------------------------------------------------------------
nginx                  UP         0Y 0W 0D  0: 4:36        0
climgr                 UP         0Y 0W 0D  0: 4:36        1
restful_api            UP         0Y 0W 0D  0: 4:36        0
fcgicpa                Down      
pnscag                 Down      
pnscdme                Down      
----------------------------------------------------------------------
Feature         Status                 Configuration
----------------------------------------------------------------------
Restful API   Disabled, UP             
PNSC          Disabled, Down

Network stats:
 eth0: RX  packets:807, TX  packets:795
 eth1: RX  packets:29, TX  packets:28

Coredump file(s): R1_climgr_67_20190925-094834-UTC.core.gz, lost+found
 
  Activated profile name: None
  Resource reservation
    Disk                : 756 MB
    Memory              : 512 MB
    CPU                 : 5% system CPU

  Attached devices
    Type              Name        Alias            
    ---------------------------------------------
    NIC               ieobc_1     ieobc            
    NIC               dp_1_33     net2             
    Disk              _rootfs                      
    Disk              /opt/var                     
    Disk              /opt/var/c                   
    Serial/shell                  serial0          
    Serial/aux                    serial1          
    Serial/Syslog                 serial2          
    Serial/Trace                  serial3          
    Watchdog          watchdog-2                   

  Network interfaces
    MAC address             Attached to interface           
    ------------------------------------------------------
    54:0E:00:0B:0C:02       ieobc_1                         
    FA:16:3E:DA:62:4F       VirtualPortGroup33              

  Guest interface
  ---
  Interface: eth1
  ip address: 172.16.1.100/24

  ---

  Guest routes
  ---
  Address/Mask                         Next Hop                          Intf.
-------------------------------------------------------------------------------

  ---

  Resource admission (without profile) : passed
    Disk space    : 756MB
    Memory        : 512MB 
    CPU           : 5% system CPU 
    VCPUs         : Not specified
</code></pre>

Another command is `show remote-management` which gives a quick overview:

<pre><code><strong>R1#show remote-management status
</strong>Remote management release version: 2017.6

----------------------------------------------------------------------
Process               Status            Uptime           # of restarts
----------------------------------------------------------------------
nginx                  UP         0Y 0W 0D  0: 3: 3        0
climgr                 UP         0Y 0W 0D  0: 3: 3        1
restful_api            UP         0Y 0W 0D  0: 3: 3        0
fcgicpa                Down      
pnscag                 Down      
pnscdme                Down      
----------------------------------------------------------------------
Feature         Status                 Configuration
----------------------------------------------------------------------
Restful API   Disabled, UP             
PNSC          Disabled, Down

Network stats:
 eth0: RX  packets:554, TX  packets:548
 eth1: RX  packets:29, TX  packets:28
</code></pre>

That’s all we have to configure on our router.

### Python

I wrote two Python 3 scripts to interact with our router. You can find them in my [Gitlab CSR1000V REST API repository](https://gitlab.com/networklessons-content/csr1000v-rest-api).

#### **GET interface**

Let’s start with a simple example. The script has two functions:

*
  * get\_token
  * get\_interface

First, we authenticate with the router using a username and password. When successful, we receive a token. We then use an HTTP GET method to retrieve information about the loopback 0 interface. Let’s run the script:

<pre><code><strong>python get-interface.py
</strong>
	We received token: VqtX5iHJpURytNQYoTF2S1zFdJIRgmAPF8ZM4v9h+gE=
	
	Here is the interface information:
	
	{
		"kind": "object#interface",
		"description": "",
		"if-name": "Loopback0",
		"proxy-arp": true,
		"subnet-mask": "255.255.255.255",
		"icmp-unreachable": true,
		"ipv6-enable": false,
		"nat-direction": "",
		"icmp-redirects": true,
		"ip-address": "1.1.1.1",
		"verify-unicast-source": false,
		"type": "loopback"
	}
</code></pre>

The output above is looking good. The router reports the configuration of the loopback 0 interface.

#### **PUT interface**

The second script uses the HTTP PUT method to reconfigure our loopback 0 interface. It has two functions:

* get\_token
* put\_interface

The get\_token function is the same as in our previous script. The put\_interface function adds a header that specifies that the payload is in JSON format. We then send the payload with our updated configuration to the router.

<pre><code><strong>python put-interface.py
</strong>We received token: VqtX5iHJpURytNQYoTF2S1zFdJIRgmAPF8ZM4v9h+gE=
The router responds with status code: 204
</code></pre>

The router responds with a 204 status code. This status code means that the server successfully fulfilled the request and that there is no additional content to send.

## Conclusion

You have now learned what a REST API is and how to use it on the Cisco CSR1000v router.

* An API is a software interface so that applications can interact with our application.
* REST is an architecture style, described in a dissertation by Roy T. Fielding.
* A REST API is an API that meets 6 constraints.
* Most REST APIs use HTTP methods and JSON or XML as the data format.
* We used the HTTP GET method to retrieve interface information and the HTTP PUT method to update its configuration.

I hope you enjoyed this lesson. Feel free to leave a comment if you have any questions.
