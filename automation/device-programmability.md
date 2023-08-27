# Device Programmability

The tools we currently use to configure and monitor our networks aren’t great for network automation.

The command-line interface (CLI) is great for us humans, but not so great for network automation. When you enter a command, there is no clear feedback. On Cisco IOS, you see an empty prompt when it accepts a command. When the command has an error, you see an error message. The output of the error message can be different depending on the command you used. Show and debug commands are also problematic because there is no uniform format. The layout and presentation of each show and debug command is different. They are easy to read for humans but **difficult to parse** with scripts and network automation software.

SNMP can be used to configure network devices, but in reality this isn’t used much. SNMP is widely in use to monitor networks.

For network automation we need:

* A programmatic interface for device configuration.
* Separation between **configuration** and **operational** data.
* Integrated error checking and recovery.

There are three network automation protocols that meet these requirements:

* **NETCONF** ([RFC 4741](https://tools.ietf.org/html/rfc4741)) – Since 2006
* **RESTCONF** ([RFC 8040](https://tools.ietf.org/html/rfc8040)) – Since 2017
* **gRPC** ([RFC 6020](https://tools.ietf.org/html/rfc6020)) – Since 2015

NETCONF is the oldest protocol. gRPC is an open source protocol from Google. RESTCONF is the latest protocol from 2017. These protocols use [YANG data models](https://networklessons.com/cisco/ccnp-encor-350-401/data-models-and-structures).

In this lesson, I’ll explain how these protocols work and you will learn how to use them to configure and monitor a router. Let’s get started!

## NETCONF

NETCONF is a protocol developed by IETF to “install, manipulate, and delete the configuration of network devices”. The goal of NETCONF is to make network automation easier. It uses **XML for data encoding and Remote Procedure Call (RPC) for messages.** It runs over SSH.

Here is an overview of the different NETCONF layers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/netconf-protocol-layers.png" alt=""><figcaption></figcaption></figure>

* We use XML as the data format:
  * The items we want to configure or retrieve from a network device are encapsulated in the \<data> tag.
  * We have different actions, for example \<get-config> to retrieve the configuration or \<edit-config> to configure the device.
  * We execute operations as remote procedure calls (RPCs). You can recognize this with the\<rpc> and \<rpc-reply> tags.
  * We run NETCONF over SSH.

Here is a full list of NETCONF operations:

| Operation        | Description                                                                                                                                |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| \<get>           | <p>Retrieve running configuration and device state<br>information.</p>                                                                     |
| \<get-config>    | <p>Retrieve all or part of a specified configuration<br>datastore.</p>                                                                     |
| \<edit-config>   | <p>The &#x3C;edit-config> operation loads all or part of a specified<br>configuration to the specified target configuration datastore.</p> |
| \<copy-config>   | <p>Create or replace an entire configuration datastore<br>with the contents of another complete configuration datastore.</p>               |
| \<delete-config> | <p>Delete a configuration datastore. The &#x3C;running><br>configuration datastore cannot be deleted.</p>                                  |
| \<commit>        | <p>The &#x3C;commit> operation instructs the device to implement the<br>configuration data contained in the candidate configuration.</p>   |
| \<lock>          | <p>The &#x3C;lock> operation allows the client to lock the<br>entire configuration datastore system of a device.</p>                       |
| \<unlock>        | <p>The &#x3C;unlock> operation is used to release a<br>configuration lock, previously obtained with the &#x3C;lock> operation.</p>         |
| \<close-session> | Request graceful termination of a NETCONF session.                                                                                         |
| \<kill-session>  | Force the termination of a NETCONF session.                                                                                                |

### Configuration

Let’s see NETCONF in action.

This is the topology I’ll use:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/r1-h1.png" alt=""><figcaption></figcaption></figure>

R1 is a CSR1000V router running IOS XE Software, Version 16.06.01.

#### **Router**

We need to enable NETCONF and create a privilege level 15 user on the router:

<pre><code><strong>R1(config)#netconf-yang
</strong><strong>R1(config)#username cisco privilege 15 secret cisco
</strong></code></pre>

NETCONF requires only a single command. It runs on TCP port 830 by default. Let’s SSH into that port:

<pre><code><strong>$ ssh cisco@172.16.1.1 -p 830 netconf
</strong></code></pre>

The router responds with a “hello” message:

```powershell
<?xml version="1.0" encoding="UTF-8"?>
<hello
  xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <capabilities>
    <capability>urn:ietf:params:netconf:base:1.0</capability>
    <capability>urn:ietf:params:netconf:capability:writeable-running:1.0</capability>
    <capability>urn:ietf:params:netconf:capability:startup:1.0</capability>
    <capability>urn:ietf:params:netconf:capability:url:1.0</capability>
    <capability>urn:cisco:params:netconf:capability:pi-data-model:1.0</capability>
    <capability>urn:cisco:params:netconf:capability:notification:1.0</capability>
  </capabilities>
  <session-id>2035438880</session-id></hello>]]>]]>
```

This hello message contains the capabilities that the router supports. We need to reply to the router with a hello message of our own. This hello message contains the capabilities that we support:

```powershell
<?xml version="1.0" encoding="UTF-8"?>
<hello xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <capabilities>
        <capability>urn:ietf:params:netconf:base:1.0</capability>
    </capabilities>
</hello>]]>]]>
```

The router won’t show anything to acknowledge our hello message but we now have an active NETCONF session. Let’s request information about the GigabitEthernet2 interface of the router:

<pre class="language-powershell"><code class="lang-powershell">&#x3C;?xml version="1.0"?>
&#x3C;rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0"xmlns:cpi="http://www.cisco.com/cpi_10/schema"
message-id="101">
   &#x3C;get-config>
        &#x3C;source>
             &#x3C;running/>
         &#x3C;/source>
          &#x3C;filter>
              &#x3C;config-format-text-cmd>
               &#x3C;text-filter-spec>
        interface GigabitEthernet2
                 &#x3C;/text-filter-spec>
              &#x3C;/config-format-text-cmd>
          &#x3C;/filter>
      &#x3C;/get-config>
<strong>&#x3C;/rpc>
</strong></code></pre>

Above, you can see the XML formatted message is embedded with the \<rpc> \</rpc> tags. I also embed our action with the \<get-config> tag. The router replies with the following message:

```powershell
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0"><data><cli-c</cmd>data><cmd>!
</cmd>nterface GigabitEthernet2
</cmd>ip address dhcp
</cmd>negotiation auto
</cmd></cli-config-data></data>
</rpc-reply>]]>]]>
```

We receive the configuration of the GigabitEthernet2 interface in XML format, embedded with the \<rpc-reply> tag. The configuration of the router is embedded within the \<data> tag. You can also see that the message-id matches the ID that I added in my request. Once we are finished, we need to close the session with the router by sending the following message:

```powershell
<?xml version="1.0" encoding="UTF-8"?>
 <rpc message-id="102" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
 <close-session />
 </rpc>
 ]]>]]>
```

The router lets us know that the session is closed:

```powershell
<?xml version="1.0" encoding="UTF-8"?><rpc-reply message-id="102" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0"><ok /></rpc-reply>]]>]]>

Connection to 172.16.1.1 closed by remote host.
```

Interacting with NETCONF manually over SSH like this works but it’s a terrible method. It’s inconvenient and, it’s easy to make errors. Instead, we should use clients or code libraries that do most of the work for us. [Ncclient](https://github.com/ncclient/ncclient) is a popular python library we can use for NETCONF.

Let’s install it:

<pre><code><strong>$ pip install ncclient
</strong></code></pre>

I created some [NETCONF sample scripts](https://gitlab.com/networklessons-labs/network-automation-orchestration) that we can run against our router. Let’s retrieve the capabilities of the router:

<pre><code><strong>$ python netconf-get-capabilities.py
</strong></code></pre>

The router replies with a list of all its capabilities:

```powershell
http://cisco.com/ns/yang/Cisco-IOS-XE-features?module=Cisco-IOS-XE-features&revision=2017-02-07&features=virtual-template,punt-num,parameter-map,multilink,l2vpn,l2,ezpm,eth-evc,esmc,efp,crypto
http://openconfig.net/yang/vlan?module=openconfig-vlan&revision=2016-05-26&deviations=cisco-xe-openconfig-vlan-deviation
http://openconfig.net/yang/network-instance-l3?module=openconfig-network-instance-l3&revision=2017-01-13
urn:ietf:params:xml:ns:yang:smiv2:RMON2-MIB?module=RMON2-MIB&revision=1996-05-27
urn:ietf:params:xml:ns:yang:smiv2:CISCO-VOICE-DIAL-CONTROL-MIB?module=CISCO-VOICE-DIAL-CONTROL-MIB&revision=2012-05-15
urn:ietf:params:xml:ns:yang:smiv2:MPLS-TE-STD-MIB?module=MPLS-TE-STD-MIB&revision=2004-06-03
urn:ietf:params:xml:ns:yang:smiv2:SNMP-TARGET-MIB?module=SNMP-TARGET-MIB&revision=1998-08-04
urn:ietf:params:xml:ns:yang:smiv2:CISCO-PTP-MIB?module=CISCO-PTP-MIB&revision=2011-01-28
urn:ietf:params:xml:ns:yang:smiv2:CISCO-VLAN-MEMBERSHIP-MIB?module=CISCO-VLAN-MEMBERSHIP-MIB&revision=2007-12-14
http://cisco.com/ns/yang/Cisco-IOS-XE-card?module=Cisco-IOS-XE-card&revision=2017-02-07
urn:ietf:params:xml:ns:yang:smiv2:CISCO-IP-TAP-MIB?module=CISCO-IP-TAP-MIB&revision=2004-03-11
```

I omitted some output above since the router produces a big list with capabilities it supports. How about retrieving the running configuration of our router? Let’s try it:

<pre><code><strong>$ python netconf-get-running-configuration.py
</strong></code></pre>

We receive the running configuration in XML output:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="urn:uuid:e82715c5-271c-44c8-9385-e13c3370f51a">
  <data>
    <native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native">
      <version>16.6</version>
      <boot-start-marker />
      <boot-end-marker />
      <service>
        <timestamps>
          <debug>
            <datetime>
              <msec />
            </datetime>
          </debug>
          <log>
            <datetime>
              <msec />
            </datetime>
          </log>
        </timestamps>
      </service>
      <platform>
        <console xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-platform">
          <output>serial</output>
        </console>
      </platform>
      <hostname>R1</hostname>
      <interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
        <interface>
          <name>GigabitEthernet1</name>
          <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
          <enabled>true</enabled>
          <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
            <address>
              <ip>10.255.1.209</ip>
              <netmask>255.255.0.0</netmask>
            </address>
          </ipv4>
          <ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip" />
        </interface>
        <interface>
          <name>GigabitEthernet2</name>
          <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
          <enabled>true</enabled>
          <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
            <address>
              <ip>172.16.1.1</ip>
              <netmask>255.255.255.0</netmask>
            </address>
          </ipv4>
          <ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip" />
        </interface>
      </interfaces>
    </native>
  </data>
</rpc-reply>
```

In the output above, we see the running configuration formatted in XML. I omitted some information since the XML output of the entire running configuration is quite long.

If I am only interested in a small part of the configuration then I can use a filter. For example, let’s only retrieve the interface configuration with another script:

<pre><code><strong>$ python netconf-get-running-configuration-filter.py
</strong></code></pre>

We now receive the XML configuration of our interfaces:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="urn:uuid:8af7b1b2-ed46-4caa-a23f-f7975cec67ec">
  <data>
    <interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
      <interface>
        <name>GigabitEthernet1</name>
        <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
        <enabled>true</enabled>
        <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
          <address>
            <ip>10.255.1.209</ip>
            <netmask>255.255.0.0</netmask>
          </address>
        </ipv4>
        <ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip" />
      </interface>
      <interface>
        <name>GigabitEthernet2</name>
        <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
        <enabled>true</enabled>
        <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
          <address>
            <ip>172.16.1.1</ip>
            <netmask>255.255.255.0</netmask>
          </address>
        </ipv4>
        <ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip" />
      </interface>
    </interfaces>
  </data>
</rpc-reply>
```

Excellent. We can retrieve configurations from the router but how about adding something to the configuration? I can use the output above as a blueprint to create a template for a new loopback interface.

{% hint style="info" %}
NETCONF is XML-based, so network vendors need to model their configuration structure for their network devices, which means that somebody (your network vendor) needs to model their configuration structure appropriately. YANG is often used for this. YANG data modules are for NETCONF what MIBs are for SNMP.
{% endhint %}

Let’s try another script to create a loopback interface on our router:

<pre><code><strong>$ python netconf-edit-config-add-loopback.py
</strong></code></pre>

The router replies with an OK message:

```powershell
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="urn:uuid:d8bd3e0e-da45-41e7-80b4-49722ed4f06f">
  <ok />
</rpc-reply>
```

We can verify whether the loopback is created with our previous script to fetch the running configuration or we can use the CLI:

<pre><code><strong>R1#show running-config interface Loopback 1
</strong>Building configuration...

Current configuration : 63 bytes
!
interface Loopback1
 ip address 1.1.1.1 255.255.255.255
end
</code></pre>

What about deleting something from the router? Let’s delete that loopback we just created with another script:

<pre><code><strong>$ python netconf-delete-config-loopback.py
</strong></code></pre>

The router replies with another OK:

```powershell
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="urn:uuid:e60f2c59-72de-483f-a934-e60de6451749">
  <ok />
</rpc-reply>
```

The interface is deleted. That’s it. I hope these examples have been useful to understand the basics of NETCONF.

## RESTCONF

RESTCONF is protocol which works similar to a REST API. It **maps a YANG specification to a RESTful interfac**e and uses the **HTTPS protocol for transport**. You can use **JSON or XML as data formats**. RESTCONF is newer than NETCONF but not a replacement. It’s more of a lightweight option for engineers who are familiar with REST APIs.

Here is an overview of the RESTCONF layers:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/restconf-protocol-stack.png" alt=""><figcaption></figcaption></figure>

In the picture above, you can see that RESTCONF uses HTTP actions (GET, POST, PUT, PATCH, DELETE) for its operations.

Below is a comparison of RESTCONF operations to NETCONF operations:

| **RESTCONF** | **NETCONF**                                 |
| ------------ | ------------------------------------------- |
| GET          | \<get>, \<get-config>                       |
| POST         | \<edit-config> (operation=”create”)         |
| PUT          | \<edit-config> (operation=”create/replace”) |
| PATCH        | \<edit-config> (operation=”merge”)          |
| DELETE       | \<edit-config> (operation=”delete”)         |

The structure of a RESTCONF URL looks like this:

> **https://**<mark style="color:red;">**address**</mark>**/**<mark style="color:blue;">**root**</mark>**/**<mark style="color:green;">**data**</mark>**/**<mark style="color:purple;">**yang-module:**</mark><mark style="color:orange;">**container**</mark>**/**<mark style="color:yellow;">**leaf**</mark>**?options**

Let me explain the highlighted parts:

* <mark style="color:red;">**address**</mark>: This is the hostname or IP address of the RESTCONF agent (network device).
* <mark style="color:blue;">**root**</mark>: The main entry point for RESTCONF requests.
* <mark style="color:green;">**data**</mark>: The RESTCONF API resource type for data.
* <mark style="color:purple;">**yang-module:**</mark><mark style="color:orange;">**container**</mark>: The base YANG data model container we want to use. The container is optional.
* <mark style="color:yellow;">**leaf**</mark>: An individual element from within the container.
* ?**options**: Optional parameters that impact the returned results.

Let me give you an example of what a RESTCONF URL looks like. To create one, we need YANG data models. You can find these in the [YANG git repository](https://github.com/YangModels/yang). Here’s the [IETF interfaces YANG data model](https://github.com/YangModels/yang/blob/master/vendor/cisco/xe/1661/ietf-interfaces.yang) to retrieve interface information from a Cisco IOS XE router.

You can scroll through the YANG file but it’s difficult to read. There are better ways to view a YANG data model. One great open source tool is [pyang](https://github.com/mbj4668/pyang). Pyang is a python YANG validator, transformer, and code generator. You can use it to validate YANG modules or convert them to other formats.

We can use pyang to look at the IETF interfaces data model in a tree format:

<pre class="language-powershell"><code class="lang-powershell"><strong>$ pyang -f tree ietf-interfaces.yang --tree-path interfaces
</strong><strong>module: ietf-interfaces
</strong><strong>  +--rw interfaces
</strong><strong>     +--rw interface* [name]
</strong>        +--rw name                        string
        +--rw description?                string
        +--rw type                        identityref
        +--rw enabled?                    boolean
        +--rw link-up-down-trap-enable?   enumeration {if-mib}?
</code></pre>

This is easier to read than a plain text YANG file. The items above give me the information I need for the URL that requests interface information:

* YANG data model (ietf-interfaces)
* YANG container (interfaces)

The URL I need to use becomes:

> **https://address/restconf/data/**<mark style="color:purple;">**ietf-interfaces:**</mark><mark style="color:green;">**interfaces**</mark>**/**<mark style="color:blue;">**interface=GigabitEthernet2**</mark>**?**<mark style="color:red;">**depth=unbounded**</mark>

### Configuration

Let’s see RESTCONF in action.

To test RESTCONF, we can use a CSR1000V router running IOS XE. Here is the topology again:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/r1-h1.png" alt=""><figcaption></figcaption></figure>

#### **Router**

Let’s start with the router. We need to know the IOS XE version so that we use the correct YANG model:

<pre><code><strong>R1#show version | include RELEASE
</strong>Cisco IOS Software [Everest], Virtual XE Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 16.6.1, RELEASE SOFTWARE (fc2)
</code></pre>

This router runs IOS XE 16.6.1.

To enable RESTCONF, we need to do three things:

* Enable RESTCONF
* Enable HTTPS server
* Create a privilege 15 user

Let’s configure this:

<pre><code><strong>R1(config)#restconf
</strong><strong>R1(config)#ip http secure-server
</strong><strong>R1(config)#username cisco privilege 15 secret cisco
</strong></code></pre>

That’s all we have to do on the router.

#### **Client**

Once again, the available YANG data models are in the [YANG git repository](https://github.com/YangModels/yang). We need to use the [Cisco IOS XE 16.6.1 YANG data models](https://github.com/YangModels/yang/tree/master/vendor/cisco/xe/1661) for this example.

Let’s see if we can retrieve the interface information from the router through RESTCONF. To do this, I can use the [IETF interfaces YANG data model](https://github.com/YangModels/yang/blob/master/vendor/cisco/xe/1661/ietf-interfaces.yang). Let’s use pyang one more time to see the tree format of this YANG data model:

<pre class="language-powershell"><code class="lang-powershell"><strong>$ pyang -f tree ietf-interfaces.yang --tree-path interfaces
</strong><strong>module: ietf-interfaces
</strong><strong>  +--rw interfaces
</strong>     +--rw interface* [name]
        +--rw name                        string
        +--rw description?                string
        +--rw type                        identityref
        +--rw enabled?                    boolean
        +--rw link-up-down-trap-enable?   enumeration {if-mib}?
</code></pre>

We use this information to construct the URL. Let’s fetch all interface information. I’ll use the Linux curl command:

<pre class="language-powershell"><code class="lang-powershell"><strong>$ curl --insecure -u "cisco:cisco" https://172.16.1.1/restconf/data/ietf-interfaces:interfaces
</strong></code></pre>

The “insecure” flag tells curl to skip certificate validation. We specify the username and password and the URL I want to use.

{% hint style="info" %}
If you use Microsoft Windows, you can use Linux tools like curl through [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10) or [Docker](https://networklessons.com/cisco/ccnp-encor-350-401/cloud-connectivity).
{% endhint %}

We receive the following XML output from the router:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces" xmlns:if="urn:ietf:params:xml:ns:yang:ietf-interfaces">
  <interface>
    <name>GigabitEthernet1</name>
    <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
    <enabled>true</enabled>
    <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
      <address>
        <ip>10.255.1.209</ip>
        <netmask>255.255.0.0</netmask>
      </address>
    </ipv4>
    <ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip" />
  </interface>
  <interface>
    <name>GigabitEthernet2</name>
    <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
    <enabled>true</enabled>
    <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
      <address>
        <ip>172.16.1.1</ip>
        <netmask>255.255.255.0</netmask>
      </address>
    </ipv4>
    <ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip" />
  </interface>
</interfaces>
```

Very nice. We receive the interface information in XML format. If I only want to receive information about a specific interface, then I can add the leaf element to the URL. Let’s retrieve only the GigabitEthernet2 configuration:

<pre><code><strong>$ curl --insecure -u "cisco:cisco" https://172.16.1.1/restconf/data/ietf-interfaces:interfaces/interface=GigabitEthernet2
</strong></code></pre>

This returns the following output:

```xml
<interface xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces"  xmlns:if="urn:ietf:params:xml:ns:yang:ietf-interfaces">
  <name>GigabitEthernet2</name>
  <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
  <enabled>true</enabled>
  <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
    <address>
      <ip>172.16.1.1</ip>
      <netmask>255.255.255.0</netmask>
    </address>
  </ipv4>
  <ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
  </ipv6>
</interface>
```

It’s also possible to only return the output of a single field. To do this, I need to add an option to the URL. For example, let’s retrieve the “enabled” status of the interface:

<pre><code><strong>$ curl --insecure -u "cisco:cisco" https://172.16.1.1/restconf/data/ietf-interfaces:interfaces/interface=GigabitEthernet2?fields=enabled
</strong></code></pre>

This returns the following output:

```xml
<interface xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces"  xmlns:if="urn:ietf:params:xml:ns:yang:ietf-interfaces">
  <enabled>true</enabled>
</interface>
```

So far so good. The curl command is OK for some quick testing but cumbersome for more advanced stuff like sending configurations to our router. We need better tools. One popular tool to work with APIs is [Google’s Postman](https://www.getpostman.com/).

Postman is often used for API development and has a nice GUI to work with. For the upcoming examples, you can import [my postman collection](https://www.getpostman.com/collections/32d186c9dc264a67f90c) so you can try them yourself.

In the global settings of Postman, I need to disable SSL certificate verification so that we don’t get any certificate errors. Keep in mind that this is no problem for a lab but not something you should do when you work on a production network. Here’s how to disable it:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/postman-ssl-certificate-verification-disabled.png" alt=""><figcaption></figcaption></figure>

To connect to the router, we need to configure authorization. In the authorization tab, I’ll select “Basic Auth” and configure the username and password of the router:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/postman-authorization-type.png" alt=""><figcaption></figcaption></figure>

Now we can try a URL. I’ll try the last URL I tested with the curl command:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/postman-get-request.png" alt=""><figcaption></figcaption></figure>

Once you hit the send button, you get the output below:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/postman-output.png" alt=""><figcaption></figcaption></figure>

Very nice. We can also use Postman to change the configuration on our router. We do this with an HTTP POST request. We need to do three things:

* Change the HTTP request to POST.
* Change the URL to /ietf-interfaces:interfaces
* Set the Content-Type to “application/yang-data+json”.

Here’s a screenshot:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/postman-post-content-type.png" alt=""><figcaption></figcaption></figure>

Let’s add a new loopback interface. I need to specify the template in JSON. In the previous examples where we did the HTTP GET requests, we received the output in XML. I used the output of the GigabitEthernet2 interface as a blueprint to create a new template. You can use one of the online [XML to JSON converters](https://codebeautify.org/xmltojson) to convert from XML to JSON.

Here’s my template:

```json
  {
      "ietf-interfaces:interface": {
          "name": "Loopback1",
          "description": "RESTCONF-TEST",
          "type": "iana-if-type:softwareLoopback",
          "enabled": true,
          "ietf-ip:ipv4": {
              "address": [
                  {
                      "ip": "1.1.1.1",
                      "netmask": "255.255.255.0"
                  }
              ]
          }
      }
  }
```

In Postman, go to the “Body” section, change the option to “raw”, select “Text” and paste the template:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/postman-post-body.png" alt=""><figcaption></figcaption></figure>

Hit the Send button. You won’t see any output but Postman will show “Status: 201 Created”.

HTTP status code 201 means that the request has been fulfilled and that one or more resources have been created. You can do another HTTP GET request to see if the loopback was created or use the CLI:

<pre><code><strong>R1#show running-config interface Loopback1
</strong>Building configuration...

Current configuration : 88 bytes
!
interface Loopback1
 description RESTCONF-TEST
 ip address 1.1.1.1 255.255.255.0
end
</code></pre>

Excellent. Our loopback interface is there. We can also delete our loopback interface from the configuration with Postman. To do this, change the request to an HTTP DELETE request and set the correct URL:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/postman-delete-request.png" alt=""><figcaption></figcaption></figure>

Hit the Send button and the interface will be deleted. You have now learned the basics of RESTCONF!

## gRPC (Google Remote Procedure Call)

gRPC is an open source RPC framework from Google. First of all, what is a Remote Procedure Call (RPC)?

Computer programs run sets of instructions (procedures) using the computer’s CPU. The computer that runs the software also processes the procedures. _**Remote**_** procedure calls run procedures on other computers connected to a network**. Once the instruction on the remote computer has been run, the results are returned to the local computer. To the program on the local computer, it is unknown that the procedure runs on a remote computer.

RPC uses a client-server model. The program on the local computer is the client, and the remote computer is the server. RPC has been a common programming technique in the Unix world. Microsoft Windows also uses RPCs, even to communicate locally between different programs.

One of the problems with RPCs is that there was no standard. There was no common framework that developers could use. Things slowly changed over the years. Around the 1990s, TCP/IP became the standard for networking, and beginning in 2000, HTTP became the standard to communicate between web servers and web browsers.

HTTP together with XML is a good combination for a framework. This combination resulted in the standardization of Simple Object Access Protocol (SOAP) and Web Services Description Language (WSDL).

As Web 2.0 merged, APIs started to play a bigger role:

* JSON replaced XML
* HTTP and JSON resulted in REST
* REST became more popular than SOAP

Nowadays, many APIs use REST.

HTTP is convenient, but it’s not the most optimal communication method. Google decided to build gRPC, a communication framework, from the ground up.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/04/grpc-client-server.png" alt=""><figcaption></figcaption></figure>

gRPC is a client-server protocol and uses [protocol buffers](https://developers.google.com/protocol-buffers/) as the **Interface Definition Language (IDL) to serialize data**. It’s a binary format. Compared to XML, Google claims it’s 3 to 10 times smaller and 20 to 100 times faster. gRPC runs over TCP and can transport multiple API calls concurrently over a single TCP session. It supports SSL/TLS for server authentication and to encrypt all data between the client and server.

Google uses gRPC internally, it’s quite popular, and there are libraries for many programming languages. You can use a gRPC server in one programming language and a gRPC client in another.

Error reporting in gRPC is better than REST-based APIs. Instead of using general HTTP status codes, gRPC has a formalized set of errors.

### Configuration

Cisco IOS XR supports gRPC. To test it, I’ll use the following topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/03/r1-xrv-h1.png" alt=""><figcaption></figcaption></figure>

R1 is a Cisco IOS XRv 9000 router.

#### **IOS XRv 9000**

Let’s start with the router. We need to enable gRPC and specify a port number:

<pre><code><strong>RP/0/RP0/CPU0:R1(config)#grpc
</strong><strong>RP/0/RP0/CPU0:R1(config-grpc)#port 57400
</strong></code></pre>

That’s all we need to configure. We do need the correct YANG data models so we can look up the exact version of our Cisco IOS XRv router:

<pre><code><strong>RP/0/RP0/CPU0:R1#show version
</strong>Thu Mar  7 13:50:29.044 UTC

Cisco IOS XR Software, Version 6.2.2
Copyright (c) 2013-2017 by Cisco Systems, Inc.

Build Information:
 Built By     : ahoang
 Built On     : Tue Jul 11 16:05:37 PDT 2017
 Build Host   : iox-ucs-031
 Workspace    : /auto/srcarchive13/production/6.2.2/xrv9k/workspace
 Version      : 6.2.2
 Location     : /opt/cisco/XR/packages/

cisco IOS-XRv 9000 () processor 
System uptime is 2 minutes
</code></pre>

This router runs IOS XRv version 6.2.2. Let’s keep this in mind when we look at the YANG data models. Before we do that, let’s look at some of the show commands on this router. We can verify whether gRPC is active or not:

<pre><code><strong>RP/0/RP0/CPU0:R1#show grpc status
</strong>Thu Mar  7 14:08:22.329 UTC
*************************show gRPC status**********************
---------------------------------------------------------------
transport                       :     grpc
access-family                   :     tcp4
TLS                             :     disabled
trustpoint                      :     NotSet
listening-port                  :     57400
max-request-per-user            :     10
max-request-total               :     128
_______________________________________________________________
*************************End of showing status*****************
</code></pre>

In the output above, you can see gRPC is active. Cisco IOS XR 6.0.0 and later support the following operations:

* **GetConfig** – Retrieves configuration data. Takes model path in JSON format as input argument. Returns configuration data in JSON format and error string.
* **MergeConfig** – Merges configuration data. Takes modeled data in JSON format as input argument. Returns error string.
* **DeleteConfig** – Deletes configuration data. Takes modeled data in JSON format as input argument. Returns error string.
* **ReplaceConfig** – Replaces configuration data. Takes modeled data in JSON format as input argument. Returns error string.
* **GetOper** – Retrieves operational (state) data. Takes model path in JSON format as input argument. Returns operational (state) data in JSON format and error string.

We can see the number of requests and responses for these operations with the following command:

<pre><code><strong>RP/0/RP0/CPU0:R1#show grpc statistics
</strong>Thu Mar  7 14:08:53.162 UTC
*************************show gRPC statistics******************
---------------------------------------------------------------
show-cmd-txt-request-recv       :     0
show-cmd-txt-response-sent      :     0
get-config-request-recv         :     0
get-config-response-sent        :     0
cli-config-request-recv         :     0
cli-config-response-sent        :     0
get-oper-request-recv           :     0
get-oper-response-sent          :     0
merge-config-request-recv       :     0
merge-config-response-sent      :     0
commit-replace-request-recv     :     0
commit-replace-response-sent    :     0
delete-config-request-recv      :     0
delete-config-response-sent     :     0
replace-config-request-recv     :     0
replace-config-response-sent    :     0
total-current-sessions          :     0
_______________________________________________________________
*************************End of showing statistics*************
</code></pre>

We haven’t used gRPC yet so all fields show a value of zero.

#### **Client**

We can use the [IOS XR gRPC python client](https://github.com/cisco-ie/ios-xr-grpc-python) to communicate with our router. Let’s install it:

<pre><code><strong>$ sudo pip install grpcio iosxr_grpc
</strong></code></pre>

To do anything with the router, we need the [YANG data models](https://github.com/YangModels/yang). Let’s start with something simple, like adding a static ARP entry to the router. We can use the [YANG Cisco-IOS-XR-ipv4-arp-cfg data model](https://github.com/YangModels/yang/blob/master/vendor/cisco/xr/622/Cisco-IOS-XR-ipv4-arp-cfg.yang) for this.

This file is difficult to read in plain text so let’s use pyang:

<pre><code><strong>$ pyang -f tree Cisco-IOS-XR-ipv4-arp-cfg.yang
</strong></code></pre>

Browsing through the data model, it seems I need to use the “arpgmp” container to add a static ARP entry. Let’s take a closer look with the “–tree-path” parameter:

<pre class="language-powershell"><code class="lang-powershell"><strong>$ pyang -f tree Cisco-IOS-XR-ipv4-arp-cfg.yang --tree-path "arpgmp"
</strong>module: Cisco-IOS-XR-ipv4-arp-cfg
  +--rw arpgmp
     +--rw vrf* [vrf-name]
        +--rw entries
        |  +--rw entry* [address]
        |     +--rw address          inet:ipv4-address-no-zone
        |     +--rw mac-address?     yang:mac-address
        |     +--rw encapsulation?   Arp-encap
        |     +--rw entry-type?      Arp-entry
        |     +--rw interface?       xr:Interface-name
        +--rw vrf-name    xr:Cisco-ios-xr-string

  augment /a1:interface-configurations/a1:interface-configuration:
    +--rw dagrs
  augment /a1:interface-configurations/a1:interface-configuration:
    +--rw ipv4arp
</code></pre>

You can use the output from the tree above to create a JSON template. However, I find it easier to configure something on the CLI, then retrieve the output through gRPC so you can use it as a template. This saves you the hassle of manually creating a JSON template that is prone to errors.

Let’s create a static ARP entry on the CLI:

<pre><code><strong>RP/0/RP0/CPU0:R1(config)#arp vrf default 1.1.1.1 0001.0001.0001 ARPA interface GigabitEthernet0/0/0/0
</strong></code></pre>

I created a script that uses the getconfig operation to fetch the YANG arpgmp container from the YANG Cisco-IOS-XR-ipv4-arp-cfg data module through gRPC. You can find it in the [git repository](https://gitlab.com/networklessons-labs/network-automation-orchestration). Let’s run it:

<pre><code><strong>$ python grpc-cisco-get-config.py
</strong></code></pre>

We receive the following output from the router:

```json
{
 "Cisco-IOS-XR-ipv4-arp-cfg:arpgmp": {
  "vrf": [
   {
    "vrf-name": "default",
    "entries": {
     "entry": [
      {
       "address": "1.1.1.1",
       "mac-address": "00:01:00:01:00:01",
       "encapsulation": "arpa",
       "entry-type": "static",
       "interface": "GigabitEthernet0/0/0/0"
      }
     ]
    }
   }
  ]
 }
}
```

This is useful. I see what I just configured on the CLI in JSON format. I can use this as a template now. Let’s copy the output above, change it a bit, and save it as a JSON file:

```json
{
    "Cisco-IOS-XR-ipv4-arp-cfg:arpgmp": {
     "vrf": [
      {
       "vrf-name": "default",
       "entries": {
        "entry": [
         {
          "address": "2.2.2.2",
          "mac-address": "00:02:00:02:00:02",
          "encapsulation": "arpa",
          "entry-type": "static",
          "interface": "GigabitEthernet0/0/0/0"
         }
        ]
       }
      }
     ]
    }
   }
```

I’ll use my second script to send this JSON template with the ReplaceConfig operation to the router.

<pre><code><strong>$ python grpc-cisco-replace-config.py
</strong>Received ConfigReply without errors
</code></pre>

The router reports that it successfully received the configuration. Let’s verify whether that is true or not. You can use the `grpc-cisco-get-config.py` script again or use the CLI:

<pre><code><strong>RP/0/RP0/CPU0:R1#show running-config | include arp
</strong>Wed Mar 13 16:47:23.227 UTC
Building configuration...
arp vrf default 2.2.2.2 0002.0002.0002 ARPA interface GigabitEthernet0/0/0/0
</code></pre>

Above, we see the ARP entry that I specified in the JSON template. We can also check the gRPC statistics on the router to verify that it received the replace config request message and responded with a replace config response:

<pre><code><strong>RP/0/RP0/CPU0:R1#show grpc statistics | include replace
</strong>Thu Mar  7 15:29:04.876 UTC
commit-replace-request-recv     :     0
commit-replace-response-sent    :     0
replace-config-request-recv     :     1
replace-config-response-sent    :     1
</code></pre>

That’s it! We successfully fetched the configuration and made a change to our configuration through gRPC.

## Conclusion

You have now learned about the basics of device programmability and how you can use NETCONF, RESTCONF, and gRPC to configure your network devices:

* The CLI isn’t suitable for network automation. SNMP is widely in use for network monitoring but not used often for network automation.
* For network automation, we need:
  * A programmatic interface for device configuration.
  * Separation between configuration and operational data.
  * Integrated error checking and recovery.
* There are three network automation protocols that meet the above requirements:
  * NETCONF
  * RESTCONF
  * gRPC
* NETCONF uses XML for data encoding and SSH for transport.
* RESTCONF is similar to a REST API. It uses JSON and XML as data format and HTTPS for transport.
* gRPC is an open source RPC framework by Google:
  * Client-server protocol and uses protocol buffers as the Interface Definition Language (IDL) to serialize data.
  * Has libraries for many programming languages.
  * Uses a formalized set of errors.

I hope you enjoyed this lesson. If you have any questions, feel free to leave a comment!
