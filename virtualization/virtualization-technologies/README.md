# Virtualization Technologies

## Cloud Deployment Models

According to the [definition](https://csrc.nist.gov/publications/detail/sp/800-145/final) of the National Institute of Standards and Technology ([NIST](https://www.nist.gov/)) there are four cloud deployment models:

* Public cloud
* Private cloud
* Community cloud
* Hybrid cloud

In this lesson, you will learn about these four cloud deployment models.

## Public Cloud

The public cloud is the cloud that most people are familiar with. The cloud infrastructure is available to everyone over the Internet. **The cloud service provider (CSP) owns, manages, and operates the public cloud.** The infrastructure is at the premises of the CSP.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/public-clouds.png" alt=""><figcaption></figcaption></figure>

You pay for the services, storage, or compute resources you use. Customers that use the public cloud use shared resources. One advantage of the public cloud is that you don’t need to buy and maintain the physical infrastructure. Your connection to the public cloud could be over the Internet or a private WAN connection.

Examples of public cloud providers:

* Amazon AWS
* Microsoft Azure
* Google Cloud
* IBM cloud
* Alibaba Cloud

## Private Cloud

The private cloud is a cloud model where a **single organization uses the cloud**.The organization or a third party could own, manage, and operate the cloud. A combination of the two is also possible. This cloud can exist on-premises or off-premises.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/private-cloud-on-or-off-premise.png" alt=""><figcaption></figcaption></figure>

Organizations that want more control over their cloud or that are bound by the law often use private clouds.

One difference between traditional virtualization and cloud computing is how we deliver services. Traditional virtualization often uses manual intervention to deliver a service. Cloud computing uses orchestration and automation to deliver a service without manual intervention.

Here are four examples:

* OpenStack
* Microsoft Azure Stack
* VMWare vCloud Suite
* Amazon AWS Outposts

Public cloud providers can also emulate a private cloud within a public cloud. We call this a **virtual private cloud**. Amazon AWS and Google Cloud call this **Virtual Private Cloud (VPC).** Microsoft Azure calls this **Virtual Network (Vnet)**. VPC and Vnet isolate resources with a virtual network unreachable from other customers.

## Community Cloud

The community cloud is a private cloud for organizations that **share common interests.** For example mission objectives, compliance policies, and security. This cloud can exist on-premises or off-premises.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/community-cloud-options.png" alt=""><figcaption></figcaption></figure>

Because of regulatory standards and the law, the government and public sector often use community clouds. There are cloud providers that offer community clouds, here are some examples:

* Amazon AWS GovCloud
* Google Apps for Government
* Microsoft Cloud for Government
* Carpathia

## Hybrid Cloud

A hybrid cloud is when we **combine a private and public cloud**. You don’t have a hybrid cloud when you use a separate private and public cloud. We call it a hybrid cloud when the **private and public cloud is integrated**. An organization might run a Microsoft Exchange server in their private cloud but use Microsoft Azure Active Directory in the public cloud for authentication.

Another example is an organization that runs most of their workloads on their private cloud. When the private cloud is at 100% capacity, they use the public cloud. We call this **cloud bursting**. The scaling from the private cloud to the public cloud is seamless, you don’t even notice whether you are on the private or public cloud.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/hybrid-cloud-topology.png" alt=""><figcaption></figcaption></figure>

Here are three examples of hybrid clouds:

* Cisco Hybrid Cloud Platform for Google Cloud
* IBM Hybrid Cloud Platform
* Rackspace Hybrid Cloud

## Multicloud

This cloud “model” is not in the NIST list of deployment models but good to know since many organizations are interested in multicloud strategies. Some organizations use more than one public cloud provider. It’s easy to get into this. One business unit might use Microsoft technology so they pick Microsoft Azure. Another business unit is into machine learning so they use some Amazon AWS services.

Cloud providers offer different services. One cloud provider might be strong in one area and weak in another.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/inter-cloud-topology.png" alt=""><figcaption></figcaption></figure>

To connect to different public cloud providers you can use an **intercloud exchange**. The intercloud exchange offers you a connection to one or more public cloud providers. This means you don’t need to use two or more private WAN connections to connect to all your public cloud providers.

Here are three reasons to use multicloud:

* **No vendor lock-in**: using the services from a single cloud provider is convenient since everything is in one place. Unfortunately, this decision makes you dependent on the cloud provider so you might want to consider using multiple cloud providers.
* **The best solution for business case**: each business unit in your organization selects the cloud provider that matches their needs.
* **Cost**: each cloud provider has a different pricing strategy. This can also be a disadvantage since most cloud providers offer volume discounts.
* **Redundancy**: there have been major outages. AWS had an S3 outage in 2017 that took down quite some websites and services.

One disadvantage of using multiple cloud providers is that you require IT staff that has knowledge of multiple cloud providers. With the number of services offered, it’s difficult to keep up with everything.

## Conclusion

You now learned about the different cloud deployment models:

* Public cloud
* Private cloud
* Community cloud
* Hybrid cloud

And some common reasons why you might want to use multicloud. I hope you enjoyed this lesson, if you have questions, please leave a comment.
