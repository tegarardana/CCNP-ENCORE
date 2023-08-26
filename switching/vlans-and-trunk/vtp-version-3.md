# VTP Version 3

In an earlier lesson I explained the [basics of VTP (version 1 and 2)](introduction-to-vlan-trunking-protocol-vtp.md). The main goal of VTP version 3 remains to synchronize VLANs but it has a number for extras. It’s been around for a while but until recent IOS versions it wasn’t supported on Cisco Catalyst Switches.

Here are some of the new additions to VTP version 3:

* **VTP primary server**: only the primary server is able to create / modify / delete VLANs. This is a great change as you can no longer “accidently” wipe all VLANs like you could with VTP version 1 or 2.
* **Extended VLANs**: you can now synchronize VLANs in the extended VLAN range (1006 – 4094).
* **Private VLANs:** if you have VLANs that are configured as private VLANs then you can synchronize them with VTPv3.
* **RSPAN VLANs:** remote SPAN VLANs can now be synchronized.
* **MST Support:** one of the problems of MST is that you had to configure each switch manually. With VTPv3, MST configurations are synchronized.
* **Authentication improvements:** VTPv3 has more secure methods for authentication.
* **VTP mode off:** If you didn’t want to use VTP for version 1 or 2 then you had to use the transparent mode. VTPv3 can be disabled globally or per interface.
* **Compatibility:** VTP version 3 is compatible with version 2, not  version 1.

I’ll walk you through each of those and show you how to configure VTP version 3. I’ll use the following topology:

![Cisco VTP Version 3 topology](https://networklessons.com/wp-content/uploads/2015/05/cisco-vtp-version-3-topology.png)

All interfaces between the switches are configured as trunks.

### Configuration

#### Basic Configuration

First we will try to enable VTP version 3 on one of our switches:

<pre><code><strong>SW1(config)#vtp version 3
</strong>Cannot set the version to 3 because domain name is not configured
</code></pre>

The domain name is now a requirement, it can’t be null. Let’s set one and try again:

<pre><code><strong>SW1(config)#vtp domain NWL
</strong>Changing VTP domain name from NULL to NWL

%SW_VLAN-6-VTP_DOMAIN_NAME_CHG: VTP domain name changed to NWL.

<strong>SW1(config)#vtp version 3
</strong></code></pre>

Let’s do the same on SW2 and SW3:

<pre><code>SW2 &#x26; SW3#
<strong>(config)#vtp domain NWL
</strong><strong>(config)#vtp version 3
</strong></code></pre>

All switches will be running in VTP server mode by default:

<pre><code><strong>SW1#show vtp status | include Operating Mode
</strong>VTP Operating Mode                : Server
</code></pre>

<pre><code><strong>SW2#show vtp status | include Operating Mode
</strong>VTP Operating Mode                : Server
</code></pre>

<pre><code><strong>SW3#show vtp status | include Operating Mode
</strong>VTP Operating Mode                : Server
</code></pre>

Being VTP server however is not enough to make changes to the VLAN database, take a look below:

<pre><code><strong>SW1(config)#vlan 100
</strong>VTP VLAN configuration not allowed when device is not the primary server for vlan database.
</code></pre>

This is new, one of the switches **has to be the primary server** in order to create / modify or delete VLANs. Let’s make SW1 our primary server:

<pre><code><strong>SW1#vtp primary
</strong>This system is becoming primary server for feature vlan
No conflicting VTP3 devices found.
Do you want to continue? [confirm]

%SW_VLAN-4-VTP_PRIMARY_SERVER_CHG: 0019.569d.5700 has become the primary server for the VLAN VTP feature
</code></pre>

As soon as I make SW1 the primary server then you’ll also see this message on the other switches:

```
SW2 & SW3#
%SW_VLAN-4-VTP_PRIMARY_SERVER_CHG: 0019.569d.5700 has become the primary server for the VLAN VTP feature
```

SW1 is now the primary server. We can verify this from SW1 or any other switch in our VTP domain:

<pre><code><strong>SW1#show vtp status | include Primary
</strong>VTP Operating Mode                : Primary Server
Primary ID                        : 0019.569d.5700
</code></pre>

<pre><code><strong>SW2#show vtp status | include Primary
</strong>Primary ID                        : 0019.569d.5700
Primary Description               : SW1
</code></pre>

<pre><code><strong>SW3#show vtp status | include Primary
</strong>Primary ID                        : 0019.569d.5700
Primary Description               : SW1
</code></pre>

SW2 and SW3 are able to confirm that SW1 is the primary server.  VTP version 3 also has a new command that allows us to see all switches in the same VTP domain:

<pre><code><strong>SW1#show vtp devices
</strong>Retrieving information from the VTP domain. Waiting for 5 seconds.

VTP Feature  Conf Revision Primary Server Device ID      Device Description
------------ ---- -------- -------------- -------------- ----------------------
VLAN         No   6        0019.569d.5700 0011.214e.d180 SW3
VLAN         No   6        0019.569d.5700 0011.bb0b.3600 SW2
</code></pre>

You can run this command on any of your switches, it will show all VTP members (not just the directly connected ones like CDP does).

Let’s see if we are able to synchronize some VLANs. We’ll start with something simple:

<pre><code><strong>SW1(config)#vlan 100
</strong><strong>SW1(config-vlan)#exit
</strong></code></pre>

Let’s create VLAN 100, it should show up on SW2 and SW3:

<pre><code><strong>SW2 &#x26; SW3#show vlan | include VLAN0100
</strong>100  VLAN0100                         active
</code></pre>

There it is! We can also synchronize VLANs in the **extended range (1006 – 4094)**. Let’s give it a try:

<pre><code><strong>SW1(config)#vlan 1234
</strong><strong>SW1(config-vlan)#exit
</strong></code></pre>

Let’s verify if it has been synchronized:

<pre><code><strong>SW1, SW2 &#x26; SW3#show vlan | include VLAN1234
</strong>1234 VLAN1234                         active
</code></pre>

No problem at all! Let’s look at some more advanced stuff.

#### Private VLANs

VTP version 3 is able to synchronize [private VLAN](https://networklessons.com/switching/private-vlan-pvlan-cisco-catalyst-switch) information. It only synchronizes the information from the VLAN database, not port information. Let’s create a private VLAN:

<pre><code><strong>SW1(config)#vlan 501
</strong><strong>SW1(config-vlan)#private-vlan community
</strong>
<strong>SW1(config)#vlan 502
</strong><strong>SW1(config-vlan)#private-vlan isolated
</strong>
<strong>SW1(config)#vlan 500
</strong><strong>SW1(config-vlan)#private-vlan primary
</strong><strong>SW1(config-vlan)#private-vlan association add 501
</strong><strong>SW1(config-vlan)#private-vlan association add 502
</strong></code></pre>

We’ll create VLAN 500 with two VLANs. VLAN 501 is a community VLAN and VLAN 502 is an isolated VLAN. Let’s see if it shows up on SW1:

<pre><code><strong>SW1#show vlan private-vlan
</strong>
Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
500     502       isolated
        501       community
</code></pre>

There we go, now let’s check if it has been synchronized to SW2 and SW3:

<pre><code><strong>SW2 &#x26; SW3#show vlan private-vlan
</strong>
Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
500     502       isolated
        501       community
</code></pre>

We see the exact same thing on SW2 and SW3. This is a nice addition to VTPv3.

#### Remote SPAN (RSPAN)

RSPAN VLANs are also a special “type” of VLANs. They can be synchronized with VTP now. Here’s an example:

<pre><code><strong>SW1(config)#vlan 600
</strong><strong>SW1(config-vlan)#remote-span
</strong><strong>SW1(config-vlan)#exit
</strong></code></pre>

Let’s check if it’s available on our switches:

<pre><code><strong>SW1#show vlan remote-span
</strong>
Remote SPAN VLANs
------------------------------------------------------------------------------
600
</code></pre>

<pre><code><strong>SW2#show vlan remote-span
</strong>
Remote SPAN VLANs
------------------------------------------------------------------------------
600
</code></pre>

<pre><code><strong>SW3#show vlan remote-span
</strong>
Remote SPAN VLANs
------------------------------------------------------------------------------
600
</code></pre>

No problem, it has been synchronized to all switches.

#### MST (Multiple Spanning-Tree)

Synchronizing [MST ](https://networklessons.com/cisco/ccnp-encor-350-401/multiple-spanning-tree-mst)is pretty useful. In the past you had to configure each switch seperately. VTP version 3 uses a seperate “feature” for MST. Take a look below:

<pre><code><strong>SW1#show vtp status
</strong>VTP Version capable             : 1 to 3
VTP version running             : 3
VTP Domain Name                 : NWL
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0019.569d.5700

Feature VLAN:
--------------
VTP Operating Mode                : Primary Server
Number of existing VLANs          : 12
Number of existing extended VLANs : 1
Configuration Revision            : 7
Primary ID                        : 0019.569d.5700
Primary Description               : SW1
MD5 digest                        : 0xC9 0x25 0xB3 0x86 0xE7 0xA1 0xE3 0xAE
                                    0xF8 0x2F 0xB9 0x7F 0x64 0xB3 0x43 0x5F


Feature MST:
--------------
VTP Operating Mode                : Transparent


Feature UNKNOWN:
--------------
VTP Operating Mode                : Transparent
</code></pre>

The default “VLAN” feature is used for the things we did before…VLANs, extended range VLANs, private VLANs and RSPAN. To synchronize MST information we have to use the “MST” feature. As you can see the VTP mode for this feature is currently transparent.

Just like the VLAN feature, we require a primary server that will create the MST configuration. You can use the same switch for this role or you can pick another one. To demonstrate this, I’ll make SW2 my primary server:

<pre><code><strong>SW2(config)#vtp mode server mst
</strong>Setting device to VTP Server mode for MST.
</code></pre>

First I change SW2 from transparent to server mode. Now we can set it to primary:

<pre><code><strong>SW2#vtp primary mst
</strong>This system is becoming primary server for feature  mst
No conflicting VTP3 devices found.
Do you want to continue? [confirm]

%SW_VLAN-4-VTP_PRIMARY_SERVER_CHG: 0011.bb0b.3600 has become the primary server for the MST VTP feature
</code></pre>

This message will also show up on SW1 and SW3:

```
SW1 & SW3#
%SW_VLAN-4-VTP_PRIMARY_SERVER_CHG: 0011.bb0b.3600 has become the primary server for the MST VTP feature
```

OK great, take a look now at the VTP status output:

<pre><code><strong>SW2#show vtp status | begin Feature MST
</strong>Feature MST:
--------------
VTP Operating Mode                : Primary Server
Configuration Revision            : 1
Primary ID                        : 0011.bb0b.3600
Primary Description               : SW2
MD5 digest                        : 0xE1 0xFE 0x40 0x19 0x4C 0x47 0x4D 0xA5
                                    0x9C 0x45 0x67 0xE3 0x9C 0xA3 0x92 0xEB
</code></pre>

You can see that this switch is now the primary server for the MST feature. Let’s make SW1 and SW3 our clients:

<pre><code>SW1 &#x26; SW3
<strong>(config)#vtp mode client mst
</strong>Setting device to VTP Client mode for MST.
</code></pre>

Everything is now in place so let’s create a configuration for MST. I’ll keep it simple:

<pre><code><strong>SW2(config)#spanning-tree mst configuration
</strong><strong>SW2(config-mst)#name MST
</strong><strong>SW2(config-mst)#revision 1
</strong><strong>SW2(config-mst)#instance 1 vlan 10,20,30
</strong><strong>SW2(config-mst)#instance 2 vlan 40,50,60
</strong><strong>SW2(config-mst)#exit
</strong></code></pre>

Normally you’d have to copy and paste the above on all your switches. We are going to synchronize it, first let’s enable MST on all switches:

<pre><code>SW1, SW2 &#x26; SW3
<strong>(config)#spanning-tree mode mst
</strong></code></pre>

Let’s verify the MST configuration:

<pre><code><strong>SW2#show spanning-tree mst configuration
</strong>Name      [MST]
Revision  1     Instances configured 3

Instance  Vlans mapped
--------  ---------------------------------------------------------------------
0         1-9,11-19,21-29,31-39,41-49,51-59,61-4094
1         10,20,30
2         40,50,60
-------------------------------------------------------------------------------
</code></pre>

It’s showing up on SW2, that makes sense since that’s where we created it. What about SW1 and SW3?

<pre><code><strong>SW1#show spanning-tree mst configuration
</strong>Name      [MST]
Revision  1     Instances configured 3

Instance  Vlans mapped
--------  ---------------------------------------------------------------------
0         1-9,11-19,21-29,31-39,41-49,51-59,61-4094
1         10,20,30
2         40,50,60
-------------------------------------------------------------------------------
</code></pre>

<pre><code><strong>SW3#show spanning-tree mst configuration
</strong>Name      [MST]
Revision  1     Instances configured 3

Instance  Vlans mapped
--------  ---------------------------------------------------------------------
0         1-9,11-19,21-29,31-39,41-49,51-59,61-4094
1         10,20,30
2         40,50,60
-------------------------------------------------------------------------------
</code></pre>

They received the MST configuration from SW2. It’s even stored in the running configuration:

<pre><code><strong>SW1#show running-config | begin mst
</strong>spanning-tree mode mst
spanning-tree extend system-id
!
spanning-tree mst configuration
 name MST
 revision 1
 instance 1 vlan 10, 20, 30
 instance 2 vlan 40, 50, 60
</code></pre>

Great, MST is working. Let’s look at some other things that VTP version 3 can do…

#### Feature Unknown

If you looked carefully at the output of show vtp status then you might have noticed that there were 3 features:

* VLAN
* MST
* UNKNOWN

This “unknown” feature is a placeholder for upcoming features that VTPv3 might use someday. Here’s the output of VTP status:

<pre><code><strong>SW1#show vtp status | begin UNKNOWN
</strong>Feature UNKNOWN:
--------------
VTP Operating Mode                : Transparent
</code></pre>

Right now you can only use the transparent mode for this, server and client are not supported. If you try to enable it then you’ll get an error:

<pre><code><strong>SW1(config)#vtp mode server unknown
</strong>Device cannot be VTP Server for unknown instances.
</code></pre>

<pre><code><strong>SW1(config)#vtp mode client unknown
</strong>Device cannot be VTP Client for unknown instances.
</code></pre>

Transparent mode does work:

<pre><code><strong>SW1(config)#vtp mode transparent unknown
</strong>Device mode already VTP Transparent for unknown instances.
</code></pre>

We’ll just have to wait to see if VTPv3 wil ever use a new feature…

#### Authentication

Authentication has slightly changed. Take a look below:

<pre><code><strong>SW1(config)#vtp password NWL ?
</strong>  hidden  Set the VTP password hidden option
  secret  Specify the vtp password in encrypted form
  &#x3C;cr>
</code></pre>

We have an option to use a hidden password. Let’s try that:

<pre><code>SW1, SW2 &#x26; SW3
<strong>(config)#vtp password NWL hidden
</strong>Setting device VTP password
</code></pre>

The password is now set to “NWL”. You can’t extract this password in clear text from the switch. Here’s how it shows up:

<pre><code><strong>SW1#show vtp password
</strong>VTP Password: 2AA31883CB1D0E65FE199ADF177F433A
</code></pre>

If you need to add another switch then you could copy and paste the above secret like this:

<pre><code><strong>SW2(config)#vtp password 2AA31883CB1D0E65FE199ADF177F433A secret
</strong>Setting device VTP password
</code></pre>

#### VTP Mode Off

VTP version 3 supports the “off” mode. The difference compared to the transparent mode is that it will be disabled 100%. Transparent mode will not synchronize itself but it will keep forwarding VTP advertisements. Here’s how to disable VTP:

<pre><code><strong>SW3(config)#vtp mode off ?
</strong>  mst      Set the mode for MST VTP instance.
  unknown  Set the mode for unknown VTP instances.
  vlan     Set the mode for VLAN VTP instance.
</code></pre>

You can disable it for the different “features”. Here’s how to disable VTP for the VLAN feature:

<pre><code><strong>SW3(config)#vtp mode off vlan
</strong></code></pre>

It is now disabled globally. You can also disable VTP on the interface level:

<pre><code><strong>SW3(config)#interface FastEthernet 1/0/21
</strong><strong>SW3(config-if)#no vtp
</strong></code></pre>

This interface will no longer participate in VTP.

#### Backward Compatibility

VTP version 3 is compatible with version 2, not with version 1. Typically when your VTPv3 switch receives a VTPv2 advertisement it should forward an advertisement that is compatible with version 2.

I tried to demonstrate this on my Cisco Catalyst 3750 switch but whatever I tried, it kept ignoring my VTP advertisements. Here’s what I tried:

<pre><code><strong>SW4(config)#vtp version 2
</strong></code></pre>

If you try this, make sure you disable the passwords on your VTPv3 switches. VTP version 2 doesn’t support the new password mechanism:

<pre><code><strong>SW1, SW2 &#x26; SW3(config)#no vtp password
</strong>Clearing device VTP password.
</code></pre>

To see what is going on between the switches you can enable the following debug:

<pre><code><strong>SW4#debug sw-vlan vtp packets
</strong>vtp packets debugging is on
</code></pre>

In my case I kept receiving this message:

```
SW4#
VTP LOG RUNTIME: Incoming packet version rcvd 3 unknown
```

My 3750 switch running IOS image `c3750-ipservicesk9-mz.122-55.SE9.bin` was unable to receive anything through VTP. I think this has something to do with this IOS version. If you are getting another result, please let me know.

Anyway that’s all I have on VTP version 3. I hope this lesson has been useful, if you have any questions then feel free to leave a comment!

\
