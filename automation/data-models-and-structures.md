# Data Models and Structures

The most common way for the last thirty years to configure our network devices is the command-line interface (CLI). **The CLI is great for humans, but not so great for computers**.

We use configuration commands to configure everything and show or debug commands to verify our work. The output of show (and debug) commands is formatted, so it’s easy to read for humans. However, it’s a pain to use the CLI for scripts or network automation tools. You have to parse a show command to get the information you want, and the output is different for each show command.

Configuration commands are also an issue. On Cisco IOS, when you enter a command, there is no confirmation of whether the router or switch accepts the command or not. You only see the empty prompt. If you paste a lot of commands, sometimes the console can’t keep up. For us humans, it’s easy to spot this and work around it. For CLI scripts or network automation tools, it’s a problem and you have to take this into account.

An alternative to the CLI to configure network devices is through application programming interfaces (APIs). **APIs use data formats to exchange information**. In this lesson, we’ll discuss the most popular data formats and structures. We’ll also take a look at YANG data models.

## Data Formats

There are two common data formats that APIs often use:

* **Extensible Markup Language (XML)**
* **JavaScript Object Notation (JSON)**

We also call these data formats “data serialization languages”. Another data format we often use for device configuration is YAML. Let’s talk about these three data formats.

### Extensible Markup Language (XML)

(XML) is a tag-based language and if you know HTML, this will look familiar.

Each item you add has to **start with < and end with >**. Here is a simple example:

```<devices>
  <router>
    <name>CSR1000V</name>
    <vendor>Cisco</vendor>
    <type>virtual</type>
  </router>
  <router>
    <name>1921</name>
    <vendor>Cisco</vendor>
    <type>hardware</type>
  </router>
</devices>
```

The output above shows the tag with two tags in it. The first tag contains information about a CSR1000V router. The second tag contains information about a 1921 router.

Let me show you the output of an actual router. Here is the output of show running-configuration as seen from the CLI:

```
hostname R1
!
ip cef
!   
interface GigabitEthernet0/1
 ip address 192.168.12.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 192.168.1.254 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!       
end
```

In XML, it looks like this:

```
R1#show running-config | format
<?xml version="1.0" encoding="UTF-8"?>
<Device-Configuration
  xmlns="urn:cisco:xml-pi">
  <version>
    <Param>15.6</Param>
  </version>
  <service>
    <timestamps>
      <debug>
        <datetime>
          <msec/>
        </datetime>
      </debug>
    </timestamps>
  </service>
  <service>
    <timestamps>
      <log>
        <datetime>
          <msec/>
        </datetime>
      </log>
    </timestamps>
  </service>
  <service operation="delete" >
    <password-encryption/>
  </service>
  <hostname>
    <SystemNetworkName>R1</SystemNetworkName>
  </hostname>
  <ip>
    <cef/>
  </ip>
  <interface>
    <Param>GigabitEthernet0/1</Param>
    <ConfigIf-Configuration>
      <ip>
        <address>
          <IPAddress>192.168.12.1</IPAddress>
          <IPSubnetMask>255.255.255.0</IPSubnetMask>
        </address>
      </ip>
      <duplex>
        <auto/>
      </duplex>
      <speed>
        <auto/>
      </speed>
      <media-type>
        <rj45/>
      </media-type>
    </ConfigIf-Configuration>
  </interface>
  <interface>
    <Param>GigabitEthernet0/2</Param>
    <ConfigIf-Configuration>
      <ip>
        <address>
          <IPAddress>192.168.1.254</IPAddress>
          <IPSubnetMask>255.255.255.0</IPSubnetMask>
        </address>
      </ip>
      <duplex>
        <auto/>
      </duplex>
      <speed>
        <auto/>
      </speed>
      <media-type>
        <rj45/>
      </media-type>
    </ConfigIf-Configuration>
  </interface>
  <end></end>
</Device-Configuration>
```

One more example.  Here is show arp with the CLI output:

<pre><code><strong>R1# show arp
</strong>Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.1.1               67   fa16.3ee0.d9a3  ARPA   FastEthernet0/0
Internet  192.168.1.2                8   fa16.3ee0.c9b2  ARPA   FastEthernet0/0
Internet  192.168.1.3                -   fa16.3ee0.a5a5  ARPA   FastEthernet0/0
</code></pre>

And in XML, it looks like this:

```r1#
<?xml version="1.0" encoding="UTF-8"?>
  <ShowArp xmlns="ODM://disk0:/spec.odm//show_arp">
    <ARPTable>
      <entry>
        <Protocol>Internet</Protocol>
        <Address>192.168.1.1</Address>
        <Age>67</Age>
        <MAC>fa16.3ee0.d9a3</MAC>
        <Type>ARPA</Type>
        <Interface>FastEthernet0/0</Interface>
      </entry>
      <entry>
        <Protocol>Internet</Protocol>
        <Address>192.168.1.2</Address>
        <Age>8 </Age>
        <MAC>fa16.3ee0.c9b2</MAC>
        <Type>ARPA</Type>
        <Interface>FastEthernet0/0</Interface>
      </entry>
      <entry>
        <Protocol>Internet</Protocol>
        <Address>192.168.1.3</Address>
        <MAC>fa16.3ee0.a5a5</MAC>
        <Type>ARPA</Type>
        <Interface>FastEthernet0/0</Interface>
      </entry>
    </ARPTable>
  </ShowArp>
```

These XML outputs are indented which makes them easier to read for us humans. **Indentation however, is not a requirement of XML**.

### JavaScript Object Notation (JSON)

(JSON) is newer than XML and has a simple syntax. JSON stores information in **key-value pairs and uses objects for its format**. It’s easier to read and has less overhead than XML.

* Objects start with { and end with }.
* Commas separate objects.
* Double quotes wrap names and strings.
* Lists start with \[ and end with ].

Let’s convert our first XML example into JSON.

XML:

```<devices>
  <router>
    <name>CSR1000V</name>
    <vendor>Cisco</vendor>
    <type>virtual</type>
  </router>
  <router>
    <name>1921</name>
    <vendor>Cisco</vendor>
    <type>hardware</type>
  </router>
</devices>
```

JSON:

```
  "devices": {
    "router": [
      {
        "name": "CSR1000V",
        "vendor": "Cisco",
        "type": "virtual"
      },
      {
        "name": "1921",
        "vendor": "Cisco",
        "type": "hardware"
      }
    ]
  }
}
```

If you look at the output above, you can see we used the tag twice in XML. In JSON, we only need a single “router” object. This is a list with two objects.

### YAML (YAML Ain’t Markup Language)

According to the [official website](https://yaml.org/), YAML is a human friendly data serialization standard for all programming languages. **YAML is a superset of JSON**. This means that a YAML parser understands JSON, but not necessarily the other way around.

YAML is easier to look at than XML or JSON.

* Key-value pairs are separated by : (colon).
* Lists begin with a – (hyphen).
* Indentation is a requirement. You have to use spaces, you can’t use tabs.
* You can add comments with a #.

We often use YAML files for configuration management because it’s easy to read and comments are useful.

Let’s compare JSON with YAML. Here’s the JSON object I showed you before:

JSON:

```
{
  "devices": {
    "router": [
      {
        "name": "CSR1000V",
        "vendor": "Cisco",
        "type": "virtual"
      },
      {
        "name": "1921",
        "vendor": "Cisco",
        "type": "hardware"
      }
    ]
  }
}
```

Here it is in YAML:

```
devices: 
 router: 
  - name: CSR1000V
    vendor: Cisco
    type: virtual
  - name: 1921
    vendor: Cisco
    type: hardware
```

YAML is easier to read without the commas and brackets of JSON. **One disadvantage of YAML is that indentation with spaces is a requirement**. It’s easy to make indentation errors where you have a space too few or too many.

{% hint style="info" %}
When you work with XML, JSON, or YAML files, you can make your life much easier if you use an editor that helps with visualization and indentation. I use Visual Studio Code for almost everything nowadays.
{% endhint %}

## Data Models

Data models describe the things you can configure, monitor, and the actions you can perform on a network device. In this section, we will discuss YANG.

### Yet Another Next Generation (YANG)

SNMP is widely used to monitor networks. You can use SNMP to configure network devices, but in reality we don’t use it much. CLI scripting is a more popular option.

YANG is a modeling language and uses data models that are **similar to to SNMP Management Information Base (MIBs)**. YANG is a standard, described in [RFC 6020](https://tools.ietf.org/html/rfc6020) and increasing in popularity. YANG uses data models that describe:

* What you can **configure** on a device.
* What you can **monitor** on a device.
* **Administrative actions** you can perform on a device like clearing interface counters or resetting the OSPF process.

YANG Data models can be:

* **Common data models**: industry-wide standard YANG models from organizations like IETF and IEEE.
* **Vendor data models**: specific models that only apply to products or features from a vendor.

These data models allow a uniform way for us to configure, monitor, and interact with network devices. Network automation tools like [NETCONF, RESTCONF, and gRPC](https://networklessons.com/cisco/ccnp-encor-350-401/device-programmability) require YANG data models.  YANG uses a hierarchical tree structure, similar to the XML data format. There is a **clear distinction between configuration data and state information**.

A YANG module defines a data model through the data of a network device, and the hierarchical organization and constraints of that data. YANG identifies each module with a namespace URL.

You can find a collection of YANG modules in the [YANG git repository](https://github.com/YangModels/yang). In this repository, there are also Cisco modules. For example, take a look at the modules for [IOX XE 16.10.1](https://github.com/YangModels/yang/tree/master/vendor/cisco/xe/16101)

There are many module files in the repository. Here is the [ARP module](https://github.com/YangModels/yang/blob/master/vendor/cisco/xe/16101/Cisco-IOS-XE-arp.yang) to create a static ARP entry:

```
module Cisco-IOS-XE-arp {
  namespace "http://cisco.com/ns/yang/Cisco-IOS-XE-arp";
  prefix ios-arp;


  import ietf-inet-types {
      prefix inet;
  }

  import Cisco-IOS-XE-native {
    prefix ios;
  }

  organization
    "Cisco Systems, Inc.";

  contact
    "Cisco Systems, Inc.
     Customer Service

     Postal: 170 W Tasman Drive
     San Jose, CA 95134

     Tel: +1 1800 553-NETS

     E-mail: cs-yang@cisco.com";

  description
    "Cisco XE Native Access Point (AP) Group Yang model.
     Copyright (c) 2018 by Cisco Systems, Inc.
     All rights reserved.";

  // =========================================================================
  // REVISION
  // =========================================================================
  revision 2018-06-28{
    description
        "Added must constraints for deleting vrf";
  }

  revision 2018-06-17{
    description
      "Add arp alias";
  }
  revision 2017-11-07 {
    description
      "Add arp vrf";
  }
  revision 2017-01-16 {
    description
      "Initial Revision";
  }

  grouping arp-entry-grouping {
    list arp-entry {
      description
        "Configure an arp entry";
      key "ip";
      leaf ip {
        description
          "IP address of ARP entry";
        type inet:ip-address;
      }
      leaf hardware-address {
        description
          "48-bit hardware address of ARP entry";
        type string;
      }
      leaf arp-type {
        type enumeration {
	  enum ARPA;
	  enum SAP;
	  enum SMDS;
	  enum SNAP;
	  enum SRP-A;
	  enum SRP-B;
	}
      }
      leaf alias {
        description
          "Respond to ARP requests for the IP address";
        type empty;
      }
    }
  }

  grouping config-arp-grouping {
    container arp {
      description
        "Set a static ARP entry";
      uses arp-entry-grouping;
      list vrf {
        description
          "Configure static ARP for a VPN Routing/Forwarding instance";
        key "vrf-name";
        leaf vrf-name {
          description
	    "VPN Routing/Forwarding instance name";
	  must "/ios:native/ios:vrf/ios:definition[ios:name=current()] or /ios:native/ios:ip/ios:vrf[ios:name=current()]" {
	    error-message "VRF must be created 1st, deleted last";
	  }
          type string;
        }
        uses arp-entry-grouping;
      }
    }
  }

  /////////////////////////////////////////////////////////
  // native
  /////////////////////////////////////////////////////////
  augment "/ios:native" {
    uses config-arp-grouping;
  } //augment
} //module
```

Above you can see that “Cisco-IOS-XE-arp” is the name of the YANG module. This module describes the data model to create static ARP entries.

The output above is difficult to read. There is a useful tool called pyang that creates an easily readable tree format. Pyang also shows all user configurable fields (rw) and state values (ro). Here is an example:

<pre><code><strong>pyang -f tree Cisco-IOS-XE-arp.yang
</strong>
module: Cisco-IOS-XE-arp
  augment /ios:native:
    +--rw arp
       +--rw arp-entry* [ip]
       |  +--rw ip                  inet:ip-address
       |  +--rw hardware-address?   string
       |  +--rw arp-type?           enumeration
       |  +--rw alias?              empty
       +--rw vrf* [vrf-name]
          +--rw vrf-name     string
          +--rw arp-entry* [ip]
             +--rw ip                  inet:ip-address
             +--rw hardware-address?   string
             +--rw arp-type?           enumeration
             +--rw alias?              empty
</code></pre>

Above, you see all the user configurable fields to create an ARP entry.

In the [device programmability lesson](https://networklessons.com/cisco/ccnp-encor-350-401/device-programmability), you can see how we use YANG to configure and monitor Cisco devices.

## Conclusion

You have now learned about data models and structures:

* The CLI is great for humans, but not so great for script and network automation.
  * Difficult to parse show commands.
  * Pasting commands is unreliable.
* APIs are an alternative to the CLI.
  * APIs use data formats (data serialization languages) to exchange information.
* XML is a tag-based language.
  * Each item starts with < and ends with >.
* JSON stores information in key-value pairs and uses objects.
  * Rules:
    * Objects start with { and end with }.
    * Commas separate objects.
    * Double quotes wrap names and strings.
    * Lists start with \[ and end with ].
  * Easier to read than XML.
  * Less overhead than XML.
* YAML is a superset of JSON.
  * Human friendly format, easy to read.
  * Rules:
    * We separate key-value pairs with : (colon).
    * Lists begin with a – (hyphen).
    * Indentation with spaces is a requirement. You can’t use tabs.
    * You can add comments with a #.
  * YAML files are popular for configuration management:
    * It’s easy to read.
    * Comments are useful.
* YANG is a modeling language, described in [RFC 6020](https://tools.ietf.org/html/rfc6020) and an alternative to SNMP.
  * YANG uses data models to describe what you can configure and monitor on network devices.
    * Data models can be common industry-wide standards or vendor specific.
  * YANG uses a hierarchical tree structure, similar to XML
    * There is a clear distinction between configuration data and state information.

I hope you enjoyed this lesson. If you have any questions, feel free to leave a comment.
