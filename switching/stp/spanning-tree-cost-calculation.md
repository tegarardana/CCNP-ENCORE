# Spanning-Tree Cost Calculation

Non-root bridges need to find the **shortest path to the root bridge.** What will happen if we have a mix of different interface types like Ethernet, FastEthernet and Gigabit? Let’s find out!

Here’s the topology I will use to explain the spanning-tree cost calculation:

\


<figure><img src="../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

In the picture above we have a larger network with multiple switches. You can also see that there are different interface types, we have Ethernet (10 Mbit), FastEthernet (100Mbit) and Gigabit (1000Mbit). SW1 on top is the root bridge so all other switches are non-root and need to find the shortest path to the root bridge.

| Bandwidth | Cost |
| --------- | ---- |
| 10 Mbit   | 100  |
| 100 Mbit  | 19   |
| 1000 Mbit | 4    |

Spanning-tree uses **cost** to determine the shortest path to the root bridge. The slower the interface, the higher the cost is. The path with the lowest cost will be used to reach the root bridge.

Here’s where you can find the cost value:

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

In the BPDU you can see a field called **root path cost**. This is where each switch will insert the **cost of its shortest path** to the root bridge. Once the switches found out which switch is declared as root bridge they will look for the shortest path to get there. **BPDUs will flow from the root bridge downwards to all switches**.

{% hint style="info" %}
If you studied CCNA or CCNP ROUTE then this story about spanning-tree cost might sound familiar. OSPF (Open Shortest Path First) also uses cost to calculate the shortest path to its destination. Both spanning-tree and OSPF use cost to find the shortest path but there is one big difference. OSPF builds a topology database (LSDB) so all routers know exactly what the network looks like. Spanning-tree is “dumb”…switches have no idea what the topology looks like. BPDUs flow from the root bridge downwards to all switches, switches will make a decision based on the BPDUs that they receive!
{% endhint %}

Here’s an example of the different spanning-tree costs for our topology:

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

SW2 will use the direct link to SW1 as its root port since this is a 100 Mbit interface and has a cost of 19. It will forward BPDUs towards SW4; in the root path cost field of the BPDU you will find a cost of 19. SW3 is also receiving BPDUs from SW1 so it’s possible that at this moment it selects its 10 Mbit interface as the root port. Let’s continue…

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

This picture needs some more explanation so let me break it down:

* SW3 receives BPDUs on its 10 Mbit interface (cost 100) and on its 1000 Mbit interface (cost 4). It will use its 1000 Mbit interface as its root port (shortest path to the root bridge is 19+19+4=42).
* SW3 will forward BPDUs to SW4. The root path cost field will be 100.
* SW4 receives a BPDU from SW2 with a root path cost of 19.
* SW4 receives a BPDU from SW3 with a root path cost of 100.
* The path through SW2 is shorter so this will become the root port for SW4.
* SW4 will forward BPDUs towards SW3 and SW5. In the root path cost field of the BPDU we will find a cost of 38 (its root path cost of 19 + its own interface cost of 19).
* SW3 will forward BPDUs towards SW5 and inserts a cost of 42 in the root path cost field (19 + 19 + 4).

The complete picture will look like this:

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

SW5 receives BPDUs from SW3 and SW4. In the BPDU we will look at the root path cost field and we’ll see the following information:

* BPDU from SW3: cost 42
* BPDU from SW4: cost 38

SW5 will add the cost of its own interface towards SW4 so the total cost to reach the root bridge through SW4 is 38 + 19 (cost of 100 Mbit interface) = 57. The total cost to reach the root bridge through SW3 is 42 + 100 (10 Mbit interface) = 142. As a result, it will select the interface towards SW4 as its root port.

Are you following me so far? Keep in mind that switches only make decisions on the BPDUs that they receive! They have no idea what the topology looks like. The only thing they do know is on which interface they received the **best BPDU.** The best BPDU is the one with the shortest path to the root bridge!

What if the cost is equal?

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Take a look at the picture above. SW1 is the root bridge and SW2 is non-root. We have two links between these switches so that we have redundancy. Redundancy means loops so spanning-tree is going to block one the interfaces on SW2.

SW2 will receive BPDUs on both interfaces but the root path cost field will be the same! Which one are we going to block? Fa0/1 or fa0/2? When the cost is equal spanning-tree will look at the **port priority.** By default the port priority is the **same for all interfaces** which means that the **interface number will be the tie-breaker.**

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

The lowest interface number will be chosen so fa0/2 will be blocked here. Of course port priority is a value that we can change so we can choose which interface will be blocked, I’ll show you later how to do this!

Whenever spanning-tree has to make a decision, it uses the following list:

1. **Lowest bridge ID**: the switch with the lowest bridge ID becomes the root bridge.
2. **Lowest path cost to root bridge**: when the switch receives multiple BPDUs it will select the interface that has the lowest cost to reach the root bridge as the root port.
3. **Lowest sender bridge ID**: when a switch is connected to two switches that it can use to reach the root bridge and the cost to reach the root bridge is the same, it will select the interface connecting to the switch with the lowest bridge ID as the root port.
4. **Lowest sender port ID**: when the switch has two interfaces connecting to the same switch, and the cost to reach the root bridge is the same it will use the interface with the lowest number as the root port.

This list is something to write down and remember: That’s all for now, I hope this helps you to understand the spanning-tree cost calculation!
