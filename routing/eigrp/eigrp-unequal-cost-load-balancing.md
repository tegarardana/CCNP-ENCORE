# EIGRP Unequal Cost Load Balancing

Just like OSPF or RIP, EIGRP can do load balancing, but it has one more trick in its hat. RIP and OSPF both can do load-balancing, but the paths have to be equal.

EIGRP can do something cool…unequal load-balancing! Even better it will share traffic in a proportional way, if you have a feasible successor that has a feasible distance that is 5 times worse than the successor, then traffic will be shared in a 5:1 way.

Let’s take a look at an example of how EIGRP can do load balancing:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/eigrp-unequal-cost-load-balancing-topology.png" alt=""><figcaption></figcaption></figure>

We’ll view this topology from R1’s perspective. Let’s fill in the successor, feasible successor, advertised, and feasible distance in a table. If you have no idea how this works, please read my[ introduction to EIGRP](https://networklessons.com/cisco/ccnp-encor-350-401/introduction-to-eigrp) first.

|    | Advertised Distance | Feasible distance |                    |
| -- | ------------------- | ----------------- | ------------------ |
| R2 | 15                  | 20                |                    |
| R3 | 10                  | 15                | SUCCESSOR          |
| R4 | 14                  | 114               | FEASIBLE SUCCESSOR |

This is our first example where we found out the successor and feasible successor. If you look at the routing table, you will only find the successor there. Now we are going to change things so we’ll see the feasible successor in the routing table as well so it will load-balance.

You can do this by using the `variance` command. The variance command works as a multiplier:

* Our successor has a feasible distance of 15.
* Our feasible successor has a feasible distance of 114.

{% hint style="info" %}
In order to load-balance, our feasible successor needs to have a feasible distance that is less or equal to the successor X multiplier.
{% endhint %}

If we set the variance at 2, this is what we get:

The feasible distance of the successor is 15 x 2 (multiplier) = 30.

114 is higher than 30, so we don’t do any load-balancing.

If we set the variance at 5, this is what we get:

The feasible distance of the successor is 15 x 5 (multiplier) = 75.

114 is still higher than 75, so there is still no load-balancing here.

Now I’m going to set the variance at 8, and this is what we get:

The feasible distance of the successor is 15 x 8 = 120.

114 is lower than 120, so now we will put the feasible successor in the routing table and start load-balancing!

Are we ever going to use the route through R2? No, we won’t since **it’s not a feasible successor**!

I hope this explanation is useful to you. Want to see this in action? Take a look at the [variance configuration example](https://networklessons.com/cisco/ccnp-encor-350-401/eigrp-variance-command-example).

If you have any questions, please leave a comment.
