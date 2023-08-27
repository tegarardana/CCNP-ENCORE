# Cisco WLC Deployment Models

Let’s say you want to deploy a WLC with multiple lightweight APs in your network. Where do you connect the WLC? In the core layer? The distribution or access layer? Or perhaps in the data center?

The [Split-MAC architecture](https://networklessons.com/cisco/ccnp-encor-350-401/cisco-wireless-network-architectures) with the WLC and APs supports **multiple deployment models**. What is essential to understand is that all data from wireless clients goes through the CAPWAP tunnel to the WLC.s

Each deployment model puts the WLC in a different location in the network. In this lesson, we’ll take a look at the various options and their pros/cons.

## Unified WLC Deployment

With the centralized WLC deployment model, the WLC is a hardware appliance in a central location in the network. This deployment is a good choice if the majority of your wireless traffic is destined to the edge of your network, like the Internet or a data center. It’s easy to enforce security policies because all traffic ends up in a central location. Another name for this deployment is **centralized WLC deployment**.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-wireless-deployment-unified-wlc.png" alt=""><figcaption></figcaption></figure>

In the picture above, the unified WLC connects to the core layer. This deployment model supports up to 6000 APs per WLC. If you need more than 6000 APs, you need to add an extra WLC.

## Cloud-based WLC Deployment

With the cloud-based WLC deployment model, we add the WLC to the data center in a private cloud. With this deployment model, the **WLC is a virtual machine**, not a physical appliance. It supports up to 3000 APs, so if you need more, you create a second virtual machine.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-wireless-deployment-cloud-wlc.png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
Don’t confuse this deployment model with [cloud-based AP architecture](https://networklessons.com/cisco/ccnp-encor-350-401/cisco-wireless-network-architectures) like Cisco Meraki. With the Cisco Meraki architecture, the management function is in the cloud and we use autonomous APs on the network.
{% endhint %}

## Embedded WLC Deployment

If you have a small number of APs, like in a small campus or branch location, then you can use an embedded WLC in switch stacks. This deployment option is **a switch with an integrated WLC**, and it supports up to 200 APs.

The APs **don’t have to be physically connected to the switch that hosts the WLC**. APs connected to other switches can join the WLC as well. When the number of APs increases, you can add additional switches with embedded WLCs.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-wireless-deployment-embedded-wlc.png" alt=""><figcaption></figcaption></figure>

## Cisco Mobility Express WLC Deployment

For small branch offices with only a few APs, even an embedded WLC deployment might be too much. For this situation, you can use a Cisco mobility express WLC deployment.

This is a **virtual WLC integrated into the AP** and supports up to 100 APs. It doesn’t support all features that a “normal” WLC supports, but for a small branch network, it might be the right choice.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2019/11/cisco-wireless-deployment-mobility-express.png" alt=""><figcaption></figcaption></figure>

## Conclusion

In this lesson, we discussed the Cisco wireless deployment models:

* The Split-MAC architecture tunnels all wireless client traffic from APs to the WLC.
* Cisco offers different deployment models for WLCs:
  * Centralized WLC deployment: Hardware appliance in a central location of the network, like the core layer. Supports up to 6000 APs and is also known as unified WLC deployment.
  * Cloud-based WLC deployment: Virtual WLC that runs in a virtual machine, supports up to 3000 APs and is usually deployed in a private cloud at the data center.
  * Embedded WLC deployment: An embedded WLC in a switch stack. Useful for smaller sites and supports up to 200 APs.
  * Cisco Mobility Express WLC deployment: Virtual WLC that runs on APs. Supports fewer features compared to the full-blown WLC. Supports up to 100 APs.

Here is an overview of all deployment models and the number of clients they support:

| **Deployment model** | **WLC location** | **Maximum APs** | **Maximum Clients** | **Use**           |
| -------------------- | ---------------- | --------------- | ------------------- | ----------------- |
| Unified              | Central          | 6000            | 64000               | Enterprise Campus |
| Cloud                | Data Center      | 3000            | 32000               | Private Cloud     |
| Embedded             | Access layer     | 200             | 4000                | Small Campus      |
| Mobility Express     | AP               | 100             | 2000                | Branch            |

I hope you enjoyed this lesson. Please leave a comment if you have any questions.
