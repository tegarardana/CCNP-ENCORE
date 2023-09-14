# NAT/PAT

## Introduction to NAT and PAT

Without network address translation (NAT) or port address translation (PAT), you probably wouldn’t be able to access the internet from your computer, or at least you’ll be the only one in the house having internet access…in this lesson, I want to give you an explanation of why and how we use NAT/PAT for Internet access.

Let’s start with a topology:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/nat-example-network.png" alt=""><figcaption></figcaption></figure>

On the left side, we have a computer on our LAN with the IP address 192.168.1.1 connected to a router. From our ISP, we got the IP address 4.4.4.4, and there’s a server on the Internet using IP address 1.2.3.4. If our computer sends something to the server what would be the source and destination IP address of the IP packet, it will send?

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/without-nat-incoming-packet-1.png" alt=""><figcaption></figcaption></figure>

The source IP address will be our computer, and the destination IP address will be the server as you can see in the IP packet in the picture above.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/without-nat-return-traffic.png" alt=""><figcaption></figcaption></figure>

Once our server responds, it will create an IP packet specifying the computer’s IP address as the destination, and the source IP address will be its own IP address.

Is there anything wrong with this example? No, it’s perfectly fine except for one detail…the IP address of the computer and the IP address on the router are private IP addresses. Private IP addresses are meant for our LANs and public IP addresses are for the Internet.

This time we will configure NAT (Network Address Translation) and see what the difference is…

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/nat-translated-traffic.png" alt=""><figcaption></figcaption></figure>

The same story, our computer is going to send something to the server, but now our router has been configured for NAT. The NAT router has been configured, so IP address 192.168.1.1 has to be translated to IP address 4.4.4.4. Here’s what happens. Our NAT router will rewrite the source IP address from 192.168.1.1 to 4.4.4.4 as you can see in the IP packet above.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/nat-return-traffic1.png" alt=""><figcaption></figcaption></figure>

The server thinks it’s talking to IP address 4.4.4.4, which is why you see this IP address as the destination in the IP packet it’s sending.

Once this IP packet reaches the router, it will look again at its NAT table, translate the IP address 4.4.4.4 back into 192.168.1.1 and send it to the computer.

The example I just showed you is called static NAT. There is a 1:1 relationship between the IP address of our computer on the LAN and the IP address we got from our ISP. So what will we do if we have more computers on our LAN? We can use something called dynamic NAT.

Dynamic NAT is different compared to static NAT because:

* You can use a pool of IP addresses to translate into.
* You can use an access-list to match the hosts on your LAN, which should be translated.

To give you an example, in our static NAT picture, we used the 4.4.4.4 IP address from the ISP to translate. Our ISP is very generous, and instead of giving us a single IP address, we get a range of IP addresses. In fact, we got the whole 4.4.4.0/24 subnet.

Besides our computer 192.168.1.1, ten other computers need Internet access. What’s going to happen now? We now have a pool of IP addresses from the ISP we can use to translate into.

Let’s discuss an example:

1. The computer with 192.168.1.1 is visiting a server on the Internet. Our NAT router will translate this IP address to the first IP address from the pool, 4.4.4.1.
2. The next computer with 192.168.1.2 is now visiting a server on the Internet. Our NAT router will translate this IP address to the second IP address from the pool, 4.4.4.2.
3. The third computer with 192.168.1.3 is also visiting something on the Internet. The NAT router will translate this IP address to the third IP address from the pool, 4.4.4.3.
4. Etc.

This is what we call dynamic NAT.

Maybe I got you puzzled…you probably have more than one device at your LAN accessing the Internet, but you only got a single IP address from your ISP. How can this work?

This is where we introduce PAT or Port Address Translation. NAT only gives us a 1:1 relationship between two IP addresses. If we have multiple computers on our LAN and only a single IP address from our ISP, we need to translate port numbers as well. This way, we can have multiple computers behind a single public IP address from the ISP. Let’s take a look at an example:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2013/02/nat-small-network-two-computers-1.png" alt=""><figcaption></figcaption></figure>

Look at the network above. We have two computers on our LAN with IP addresses 192.168.1.1 and 192.168.1.2. Our router is configured for NAT:

The following situation is happening:

1. A computer with IP address 192.168.1.1 is going to connect to the server.
2. Our NAT router will translate 192.168.1.1 to 4.4.4.4.
3. Our other computer with IP address 192.168.1.2 is also connecting to the server.
4. Our NAT router now has a problem since 192.168.1.1 is already translated to 4.4.4.4. You can’t have two IP addresses translated.

This is where PAT kicks in. With PAT this is what will happen:

1. A computer with IP address 192.168.1.1 is going to connect to the server.
2. Our NAT router will translate 192.168.1.1 to 4.4.4.4 and keep track of the source and destination ports!
3. Our other computer with IP address 192.168.1.2 is also connecting to the server.
4. Since our NAT router also does PAT, it will translate 192.168.1.2 to 4.4.4.4 as well and use another source port number.

And that’s how you can have multiple computers on your LAN and make all of them access the Internet behind a single public IP address from your ISP.

The server thinks it’s only talking to 4.4.4.4, so it has no idea there is a computer with IP address 192.168.1.1 or 192.168.1.2. Does this mean NAT or PAT is a security protocol? This is a big debate, but in my opinion, it’s no security mechanism. Not seeing the true hosts on your LAN doesn’t mean you cannot connect. When your router is doing network and/or port address translation, those hosts are reachable. Security is something you implement by using access-lists, firewalls, intrusion prevention systems and security policies.

Since NAT and/or PAT are changing the IP packet, there are some applications that don’t work too well with this translation of IP addresses and ports. IPSec is an example. FTP is also troublesome behind a NAT router.

That’s everything I wanted to share about NAT/PAT for now. I hope this is useful to you! In another lesson, we’ll look at the configuration of NAT/PAT on some Cisco IOS routers.
