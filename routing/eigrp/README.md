# EIGRP

## Introduction to EIGRP

In this lesson, we’ll take a look at EIGRP (Enhanced Interior Gateway Routing Protocol), which is Cisco’s routing protocol. If you are unfamiliar with distance vector and RIP, I highly recommend reading my [Introduction to RIP](https://networklessons.com/cisco/ccie-routing-switching/rip-distance-vector-routing-protocol) first before continuing.

EIGRP stands for Enhanced Interior Gateway Routing Protocol and is a routing protocol created by Cisco. Originally, it was only available on Cisco hardware but for a few years, it’s now an [open standard](https://tools.ietf.org/html/rfc7868). EIGRP is called a hybrid or advanced distance vector protocol, and most of the rules that apply to RIP also apply here:

* Split Horizon
* Route Poisoning
* Poison Reverse

EIGRP routers will start sending hello packets to other routers just like OSPF does, if you send hello packets and you receive them you will become neighbors. EIGRP neighbors will exchange routing information which will be saved in the topology table. The best path from the topology table will be copied into the routing table:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-tables.png" alt=""><figcaption></figcaption></figure>

Selecting the best path with EIGRP works a bit differently than other routing protocols, so let’s see it in action:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-demo-3-routers-1.png" alt=""><figcaption></figcaption></figure>

We have three routers named R1, R2, and R3. We are going to calculate the best path to the destination, which is behind R3.

EIGRP uses a rich set of metrics, namely bandwidth, delay, load, and reliability, which we will cover later. These values will be put into a formula, and each link will be assigned a metric. The lower these metrics, the better.

In the picture above I have assigned some values on the interfaces, if you would look at a real EIGRP router you’ll see the numbers are very high and a bit annoying to work with. R3 will advertise to R2 its metric towards the destination:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-advertised-distance.png" alt=""><figcaption></figcaption></figure>

Basically, R3 is saying to R2: “It costs me 5 to get there”. This is called the advertised distance. R2 has a topology table, and in this topology table it will save this metric, the advertised distance to reach this destination is 5.

{% hint style="info" %}
The advertised distance is also called the **reported distance**.
{% endhint %}

We are not done yet since there is something else that R2 will save in its topology table. We know the advertised distance is five since this is what R3 told us. We also know the metric of the link between R2 and R3 since this is directly connected. R2 now knows the metric for the total path to the destination, this total path is called the feasible distance, and it will be saved in the topology table:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-feasible-distance.png" alt=""><figcaption></figcaption></figure>

You have now learned two important concepts of EIGRP. The advertised distance, your neighbor tells you how far it is for him to reach the destination, and the feasible distance which is your total distance to get to the destination.

Let’s continue! R2 is sending its feasible distance towards R1, which is 15. R1 will save this information in the topology table as the advertised distance. R2 is “telling” R1 the distance is 15:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-advertised-feasible-distance.png" alt=""><figcaption></figcaption></figure>

R1 now knows how far the destination is away for R2, and since we know the metric for the link between R1 and R2, it can also calculate the total distance, which is called the feasible distance, this information is saved in the topology table.

Are you following me so far? Let me describe these terms once again but in plain English:

* Advertised distance: How far the destination is away for your neighbor.
* Feasible distance: The total distance to the destination.

{% hint style="info" %}
The best path to the destination is called the **successor**!
{% endhint %}

The successor will be copied from the topology table to the routing table.

### Feasible Successor

With EIGRP, however, it’s possible to have a backup path which we call the feasible successor. How do we find out if we have a feasible successor? Let’s find out:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-path-calculation-example-1.png" alt=""><figcaption></figcaption></figure>

In the example above, we have a couple of routers running EIGRP; we are sitting on the router without a name on the left side and would like to know two things:

* Which path is the successor (the best path)?
* Do we have any feasible successors? (backup paths)

Let’s fill in the following table to find out:

|    | Advertised Distance | Feasible distance |   |
| -- | ------------------- | ----------------- | - |
| R1 |                     |                   |   |
| R2 |                     |                   |   |
| R3 |                     |                   |   |

If you want to try your new-learned EIGRP skills, try to fill in the advertised and feasible distance by yourself in the table above.

R1 is telling us the destination is 10 away, R2 tells us it’s 5 away, and R3 tells us it’s 9 away. We can now fill in the advertised distance part of the table: \


|    | Advertised Distance | Feasible distance |   |
| -- | ------------------- | ----------------- | - |
| R1 | 10                  |                   |   |
| R2 | 5                   |                   |   |
| R3 | 9                   |                   |   |

Since we know our directly connected links, we can add this to the advertised distance, and we’ll have our feasible distance.

|    | Advertised Distance | Feasible distance |   |
| -- | ------------------- | ----------------- | - |
| R1 | 10                  | 15                |   |
| R2 | 5                   | 10                |   |
| R3 | 9                   | 109               |   |

The path with the lowest feasible distance will be the successor (R2), so now we answered the first question.

|    | Advertised Distance | Feasible distance |           |
| -- | ------------------- | ----------------- | --------- |
| R1 | 10                  | 15                |           |
| R2 | 5                   | 10                | SUCCESSOR |
| R3 | 9                   | 109               |           |

You will find the successor in the routing table.\


To answer the second question, “do we have a feasible successor (backup path)?” we need to learn another formula:

{% hint style="info" %}
Advertised distance of feasible successor < Feasible distance of successor.
{% endhint %}

Or in plain English:

A router can become a backup path if it is closer to the destination than the total distance of your best path.

I think that sounds a bit better right? Let’s try it and see if R1 or R3 is suitable as a backup path:

The advertised distance of R1 is 10, which is equal to the feasible distance of R2, which is also 10. It has to be lower…equal is not good enough, so R1 will NOT be a feasible successor.

The advertised distance of R3 is 9, which is lower than the feasible distance of R2, which is 10. R3 will be a valid feasible successor and used as a backup path!

|               | Advertised Distance | Feasible distance |                    |
| ------------- | ------------------- | ----------------- | ------------------ |
| R1            | 10                  | 15                |                    |
| <p>R2<br></p> | 5                   | 10                | SUCCESSOR          |
| R3            | 9                   | 109               | FEASIBLE SUCCESSOR |

Excellent, so R2 is our successor, and R3 is a feasible successor. You will find both entries in the EIGRP topology, but you will only find the successor in the routing table. If you lose the successor because of a link failure, EIGRP will copy/paste the feasible successor into the routing table. This is what makes EIGRP a FAST routing protocol…but only if you have a feasible successor in the topology table.

Now look closely at the feasible distance of R3 and R1…what do you see? The metric for R3 is FAR worse than the one for R1. Does this make any sense? Did the Cisco EIGRP engineers make a horrible mistake here by using non-optimal backup paths?

Nope, this is perfectly the way it should be! Keep in mind EIGRP at heart is a distance vector protocol. It doesn’t know what the complete network looks like…it’s not a link-state routing protocol like OSPF, which DOES have a complete map of the network. Distance vector routing protocols only know which way to go (vector) and how far away the destination is (distance).

The formula you just witnessed to determine EIGRP’s feasible successors is how EIGRP can guarantee you that the backup path is 100% loop-free! I know this isn’t easy to grasp by reading text so let’s do another example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-successor-rule-example.png" alt=""><figcaption></figcaption></figure>

Keep in mind that EIGRP is by nature a distance vector routing protocol, we see the topology, but EIGRP does not!

We are looking at the EIGRP topology table of R3, and we want to reach the destination behind R4. Let’s fill in the table (try it yourself if you want a good exercise):

|    | Advertised Distance | Feasible distance |   |
| -- | ------------------- | ----------------- | - |
| R4 |                     |                   |   |
| R1 |                     |                   |   |
| R2 |                     |                   |   |

1. R4 will advertise the destination network to R3.
2. R3 will advertise the network to R1 and R2.
3. R1 will advertise the network to R2.
4. R2 will advertise the network to R1.
5. R1 will advertise this network back to R3.
6. R2 will advertise this network back to R3.

|    | Advertised Distance | Feasible distance |   |
| -- | ------------------- | ----------------- | - |
| R4 | 5                   |                   |   |
| R1 | 25                  |                   |   |
| R2 | 19                  |                   |   |

Here we have the advertised distance, our neighbors are telling us how far it is for them to reach the destination network. The next step is to fill in the feasible distances.

How did I get the numbers in the advertised distance table? Let’s look at all the routers:

R4 is easy. The destination has a distance of 5, as seen in the topology picture. This will be advertised to R3 and placed in its topology table.

R1 will learn the destination network through R3 and R2. R3 will advertise a distance of 5+4 = 9 to R1. R1 would re-advertise the distance to R3 as 3+5+4 = 12. So why didn’t I place “12” in the advertised distance field in the table? Good question! Remember split-horizon? Don’t advertise to your neighbor whatever you learned from them….R1 is not sending information about this network back to R3. To be more specific: whatever you learn on an interface, you don’t advertise back out of the same interface.

How did I get to 25? Let’s break it down:

R4 will advertise a distance of 5 toward R3. R3 will advertise 5+4 = 9 towards R2.

R2 will advertise 5+4+9 = 18 towards R1. Finally R1 will advertise 5+4+9+7 = 25 towards R3. Split Horizon doesn’t apply here since R1 learned about the destination on another interface (R2).

The same thing applies to the advertised distance of 19 for R2:

1. R4 advertises a distance of 5 to R3.
2. R3 advertises a distance of 5+4 = 9 to R1.
3. R1 advertises a distance of 5+4+3 = 12 to R2.
4. R2 advertises a distance of 5+4+3+7 = 19 to R3.

|    | Advertised Distance | Feasible distance |   |
| -- | ------------------- | ----------------- | - |
| R4 | 5                   | 9                 |   |
| R1 | 25                  | 28                |   |
| R2 | 19                  | 28                |   |

R3 has learned the advertised distance from its neighbors and knows about its own directly connected interfaces so you can fill in the feasible distance. The last step is to pick our successor.

|    | Advertised Distance | Feasible distance |           |
| -- | ------------------- | ----------------- | --------- |
| R4 | 5                   | 9                 | SUCCESSOR |
| R1 | 25                  | 28                |           |
| R2 | 19                  | 28                |           |

R4 has the lowest feasible distance, so it will become the successor…excellent! Let’s do the feasible successor check and see if there is a backup path:

{% hint style="info" %}
Advertised distance of feasible successor < Feasible distance of successor.
{% endhint %}

The advertised distance of R1 (25) and R2 (19) is higher than the feasible distance of R4 (9), so they won’t become feasible successors. This makes sense, right? If these routers become backup paths, we would have a loop!

If your neighbor is closer to the destination than your total path, you at least know it’s not getting to the destination by sending packets through your router. Perhaps it’s not the best path, but it’s absolutely 100% loop-free!

### Metrics

The next thing I want to show you are the EIGRP metrics. You have learned that RIP uses hop count, OSPF has a cost, and EIGRP has yet something else for metrics…actually four things:

* Bandwidth
* Delay
* Load
* Reliability

Bandwidth and Delay are static values; a FastEthernet link is 100 Mbit and has a delay of 100usec. Ethernet, for example, is 10 Mbit and has a delay of 1000usec. These values don’t change.

Load is how busy your link is, and reliability is how reliable your link is by looking at errors. Load and reliability are dynamic values because they can change over time.

By default EIGRP only uses bandwidth and delay. Load and reliability are disabled by default. This makes sense because we want routing protocols to be as quiet and reliable as can be. You don’t want EIGRP routers sending updates every now and then because a link is suddenly busy. Routing should be predictable, and after picking the best path, you want EIGRP only to send hello packets to maintain neighbors…nice and quiet.

That’s all I wanted to explain for now. In another lesson, we’ll take a look at EIGRP unequal cost path load balancing and the configuration of EIGRP! If you enjoyed this lesson, please leave a comment!
