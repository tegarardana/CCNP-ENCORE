# Virtualization Function (NFVI,VNF)

**A network function (NF)** is a function that is performed by a physical appliance like a router, switch, firewall, load balancer, IDS, WAN optimizer, etc. Most vendors have proprietary solutions in which hardware and software are tightly coupled. Cisco is a good example. They sell their routers and switches as appliances. You need the hardware and software. You cannot run Cisco IOS (officially) without the hardware and you can’t run anything else besides Cisco IOS on a router or switch.

In this lesson, you will learn about _virtual_ network functions and the ETSI NFV architectural framework.

## Virtual Network Function (VNF)

Nowadays, we can also use virtual solutions. A **virtual network function (VNF)** is the virtual version of a hardware device’s network function. VNFs are available as virtual machines or containers.

Here are examples of Cisco VNFs:

* vEdge Cloud
* CSR1000v
* ASAV
* Cloud Services Platform (CSP) 2100
* XRv 9000
* Firepower NGFWv
* Web Security Virtual Appliance (WSAv)
* Email Security Virtual Appliance (ESAv)
* Advanced Malware Protection Virtual (AMPv)

The advantages of VNFs are similar to server virtualization and cloud computing. We have a **shorter time to market (TTM)** because, without the hardware requirement, we can quickly launch a new network function as a virtual machine or container.

**We can scale up or down, in or out, on-demand.** Since we don’t need physical appliances, we require less physical space and less power consumption. This results in a **reduced operator capital expenditure (CAPEX) and reduced operational expenditure (OPEX)**.

## ETSI NFV Architectural Framework

The European Telecommunications Standards Institute (ETSI) created the **Network Functions Virtualization (NFV)** framework which describes standards to decouple network functions from proprietary hardware appliances and instead, **run them in software on standard hardware**.

Vendors offer VNFs and service providers choose the VNFs that they need. If we want to run VNFs from different vendors on a single open platform, we need a standard that describes how to manage, monitor, and configure our VNFs.

The framework also describes how to manage and orchestrate VNFs. Here is a picture of the framework:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/etsi-nfv-framework.png" alt=""><figcaption></figcaption></figure>

There are four components:

* **Virtualized Network Functions (VNFs)**
* **Network Functions Virtualization Infrastructure (NFVI)**
* **NFV Management and Orchestration (MANO)**
* **Operations and Billing Support System (OSS/BSS)**

Let’s walk through all of these components.

### NFV Infrastructure (NFVI)

NFV infrastructure (NFVI) is all the hardware and software we need to create a platform where we can run VNFs on. Many VNFs are available as virtual machines, so this is where we find our physical servers, storage, networking, and the virtualization layer (hypervisors).

### Virtualized Network Functions (VNFs)

This component is where we find our VNFs. The **Element Manager (EM)** is responsible for network management of one or more VNFs. The EM can also be a VNF.

### NFV Management and Orchestration (MANO)

The NFV Management and Orchestration (MANO) component has three items:

* **NFV Orchestrator (NFVO)**: the orchestrator launches, maintains, and scales VNFs. The orchestrator also validates and authorizes resource requests from the NFVI component.
* **VNF Manager (VNFM)**: the VNFM performs life-cycle management (launch, maintain, and teardown) of VNFs.
* **Virtualized Infrastructure Manager (VIM)**: the VIM manages and controls the hardware and virtualization resources in the NFVI. It collects performance metrics, fault information, and also performs life-cycle management of all NFVI resources.

The VIM is also responsible for **VNF service chaining**. Server chaining means we chain multiple VNFs together to create a service or solution. Here is an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/cisco-csr1000v-asav-service.png" alt=""><figcaption></figcaption></figure>

The CSR1000v router and ASAv are two VNFs we chain together to create a single solution. We might use the router for DMVPN and the ASAv for our firewall rules.

### Operations and Billing Support System (OSS/BSS)

The **Operational and Billing Support System (OSS/BSS)** component has the Operations **Support Systems (OSS)** and **Billing Support Systems (BSS)**. The OSS supports management functions like network inventory, management, and configuration. The BSS deals with customer management and includes systems for taking orders, payments, etc.

The acronyms OSS and BSS are often used interchangeably or abbreviated as OSS/BSS.

## Cisco NFVI

Cisco has an NFVI solution called [Cisco NFVI ](https://www.cisco.com/c/en/us/solutions/service-provider/network-functions-virtualization-nfv-infrastructure/index.html) and is based on the ETSI NFV framework. NFVI uses a combination of Cisco products and Red Hat (RHOL) OpenStack. Here is an overview:

* NFVI
  * Hardware
    * Compute: UCS servers
    * Storage: UCS Storage Servers
    * Network: Nexus switches
  * Virtual
    * Compute: RHEL KVM
    * Storage: RHEL CEPH
    * Network: OVS, VRF, SR-IOV
* VNFs
  * All Cisco VNFs we discussed in the VNF section of this lesson.
  * Third party VNFs
* MANO
  *
    * NFVO
      * Cisco NSO
    * VNFM
      * Cisco ESC
    * VIM
      * RHEL OSP
      * Cisco VIM Life-cycle Manager
      * Cisco ACI
      * Cisco VTS

With the Cisco NFVI solution, you can use Cisco products or third-party products. This applies to both the VNFs and other components like the VNF manager or VIM.

## Conclusion

In this lesson, you learned what network functions (NF) and virtual network functions (VNF) are. We also discussed the ETSI NFV architectural framework that was created to standardize a platform to run VNFs from different vendors on generic shared hardware. You learned about all the components of the NFV framework, and we talked about the Cisco NFVI solution.

I hope you enjoyed this lesson, if you have any questions, please leave a comment.
