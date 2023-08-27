# BGP Single/Dual Homed and Multi-homed Designs

When talking about ISPS, BGP, and connections, sometimes you will hear terminology like “single homed”, “dual homed”,”single multi-homed” or “dual multi-homed”. These are different design topologies where we describe how a customer is connected (using BGP) to one or more ISPs.

Let’s take a look at some examples.

## Single Homed

The single homed design means you have a single connection to a single ISP. With this design, you don’t need BGP since there is only one exit path in your network. You might as well just use a static default route that points to the ISP.

The advantage of a single-homed link is that it’s cost effective, the disadvantage is that you don’t have any redundancy. Your link is a single point of failure but so is using a single ISP.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/03/single-homed-connection.png" alt=""><figcaption></figcaption></figure>

## Dual Homed

The dual homed connection adds some redundancy. You are still only connected to a single ISP, but you use two links instead of one. There are some variations for this design. Here’s the first one:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/03/dual-homed-connection-single-routers.png" alt=""><figcaption></figcaption></figure>

With this design, we use a single router on both ends, but we do have redundant links.

To increase redundancy, we can add a second router:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/03/dual-homed-connection-two-isp-routers.png" alt=""><figcaption></figcaption></figure>

In the example above, the ISP has a second router. We also could have used a second router at the customer’s side and a single router at the ISP. For even more redundancy, add a second router at both sides:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/03/dual-homed-router-redundancy.png" alt=""><figcaption></figcaption></figure>

The example above offers the most redundancy when you are connected to a single ISP. We have two links and two routers on both ends. One disadvantage of this design is that we are still using a single ISP.

## Single Multi-homed

Multihomed means we are connected to at least two different ISPs. The most simple design looks like this:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/03/single-multihomed-connection-two-isps.png" alt=""><figcaption></figcaption></figure>

Above you see that we have a single router at the customer, connected to two different ISPs. The single point of failure in this design is that you only have one router at the customer. When it fails, you won’t be able to connect to any ISP. We can improve this by adding a second router:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/03/single-multihomed-connection-two-isps-redundancy.png" alt=""><figcaption></figcaption></figure>

This is a pretty good design, we only use single links, but we are connected to two different ISPs using different routers.

## Dual Multihomed

The dual multihomed designs means we are connected to two different ISPs, and we use redundant links. There are some variations, here’s the first one:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/03/dual-multihomed-two-isps.png" alt=""><figcaption></figcaption></figure>

Above you can see that we are connected to two different ISPs, using one router and two links to each ISP. We have redundant ISPs and links, but the router is still a single point of failure. We can improve this by adding a second router:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/03/dual-multihomed-connection-two-enterprise-routers.png" alt=""><figcaption></figcaption></figure>

The design above is better; it has two customer routers. One disadvantage, however, is that once one of your router fails, you will lose the connection to one of the ISPs. Using the same number of routers and links, the following design might be better:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/03/dual-multihomed-connection-dual-routers.png" alt=""><figcaption></figcaption></figure>

This design has redundant ISPs, routers, and links. Both customer routers are connected to both ISPs. This design does offer the highest redundancy but it’s also an expensive option.

## Conclusion

You have now learned what the different (BGP) connection options to an ISP are:

* **Single homed**: you are connected to a single ISP using a single link.
* **Dual homed**: you are connected to a single ISP using dual links.
* **Single multi-homed**: you are connected to two ISPs using single links.
* **Dual multi-homed**: you are connected to two ISPs using dual links.
