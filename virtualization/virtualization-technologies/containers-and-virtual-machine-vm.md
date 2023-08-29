# Containers and Virtual Machine (VM)

In this lesson, we’ll take a look at the differences between virtual machines and containers.

### Virtual Machines

Back in the days, a physical server would only run a single operating system with one or a few applications. We started with CPUs with a single core, then hyperthreading, and then multiple cores. The amount of memory in a single physical server also increased and became more affordable.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2017/02/physical-server-concept.png" alt=""><figcaption></figcaption></figure>

One of the reasons virtualization is now so popular is that servers were under-utilized. We can easily run multiple operating systems and multiple applications on a single physical server. A virtual machine is pretty identical to a physical server except it’s virtual. We have virtual hardware (CPUs, memory, storage, etc.) which runs on a **hypervisor**.

The hypervisor is the server virtualization software that runs on the physical server. This is where we create virtual machines and configure how much CPU cores, memory, storage, etc. each virtual machine has. There are two hypervisor types:

* **Type 1:** this hypervisor runs directly on hardware, which means you can assign more resources to virtual machines. Examples are VMware ESXi, Citrix Xen, KVM, and Microsoft Hyper-V.
* **Type 2**: this hypervisor runs on top of an operating system like Microsoft Windows, Apple MacOS, or Linux. We usually use a type 2 hypervisor on desktops or laptops. Two popular hypervisors are Oracle VM Virtualbox and VMWare Workstation. If you have never played with virtual machines before, give Virtualbox a try. You can download it for free.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/hypervisor-type-1-type-2.png" alt=""><figcaption></figcaption></figure>

Even a type 1 hypervisor has something of an OS. For example, VMWare ESXi uses a proprietary kernel called the VMkernel. It’s very lightweight compared to a “full” operating system like Microsoft Windows or any Linux distribution.

When the hypervisor or the underlying physical server crashes, all your virtual machines disappear. Fortunately, there are products so you can migrate virtual machines from one physical server to another with no downtime or noticeable latency.

One advantage of virtual machines is that we are familiar with physical servers. It’s easy to understand, it’s a server, but virtual. We can use all the management and security tools we know to manage our physical or virtual servers.

One disadvantage of virtual machines is that there is a lot of overhead. For example, let’s say I want to run the [freeradius](https://freeradius.org/) application. A virtual machine with the Ubuntu 18.04 LTS operating system and nothing else installed is already \~3400 MB. Freeradius and its dependencies require about 5 MB. This brings my total storage to 3400 + 5 = 3405 MB.

That’s 3405 MB of storage only to run a simple application. Starting a virtual machine can take minutes, it has to boot the OS and start your applications

## Containers

Containers are sometimes called _light-weight virtual machines_, but they are nothing like virtual machines.

A container is a “package” that only contains an application and its dependencies, nothing more. This package is stored as a container image. Containers run on top of a **container engine**, which runs on top of an operating system.  Starting a container is very quick since the operating system is already running. Containers are isolated from each other by the container engine.

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/container-virtualization-stack.png" alt=""><figcaption></figcaption></figure>

Containerization has many advantages:

* **Small**: the container image only has the application and its dependencies, nothing more. This results in a small container image. For example, I created a [container image with freeradius.](https://hub.docker.com/r/networklessons/docker-alpine-freeradius) It’s only \~ 5 MB and has everything you need to run freeradius.
* **Fast**: you don’t have to start a virtual server with a virtual BIOS and virtual operating system. Spinning up a container is as fast as starting an application and only takes milliseconds.
* **Portability**: The container image has everything the application needs. I can create a container image on my local machine, then ship somewhere else like a remote server. If it runs on my computer, it will run on other computers or servers.
* **Isolation**: containers run on the same operating system but are isolated from each other. When one container crashes, it doesn’t affect the other containers.
* **Scalability**:  You can add more identical containers when you need to scale out. Since containers are small and start quickly, you can scale out and in easily.
* **Immutability**:  container images are built based on a “blueprint”. The freeradius image I mentioned earlier is a [Docker](https://www.docker.com/) container; the blueprint is called a [Dockerfile](https://hub.docker.com/r/networklessons/docker-alpine-freeradius/dockerfile). When you change your code, you build a new container image.

Are there any disadvantages to containers? Containers are isolated from each other on the process level which could be less secure than a virtual machine which is fully isolated.

Another issue is security. Containers are based on a blueprint which is basically like a snapshot. You build the container image from the blueprint, and the container doesn’t change anymore. If you want to update your container, you rebuild a new container image. This is different from a (virtual) server which you configure to install the latest security updates automatically.

Be careful that you run no container images that are outdated and might have vulnerabilities. For example, take a look at the [Apache container images](https://hub.docker.com/\_/httpd?tab=tags) on Docker hub:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/docker-apache-vulnerabilities.jpg" alt=""><figcaption></figcaption></figure>

The container image with the “alpine” tag has vulnerabilities. We can click on the tag to see the vulnerability:

<figure><img src="https://cdn.networklessons.com/wp-content/uploads/2018/12/docker-apache-vulnerabilities-lua.jpg" alt=""><figcaption></figcaption></figure>

This vulnerability could be dangerous. It might also be unwise to run a public container image on your production network. When you build your own container images, you need to ensure you check them for security vulnerabilities and rebuild them when needed.

Docker is the most popular container platform. Other options:

* rkt (rocket)
* LXD
* Linux VServer
* Windows Containers

If you never played with containers, try Docker. You can download it for free, and it runs on Microsoft Windows, Linux, and Apple MacOS. [Docker Hub](https://hub.docker.com/) has plenty of container images to test.

## Conclusion

Virtual machines and containers both have advantages and disadvantages. Here is an overview of the differences between the two:

|              | Virtual Machine                                                   | Container                                                                                 |
| ------------ | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Overhead     | A lot of overhead since you we virtualize all hardware.           | Almost none since we run the container directly on the host OS with the container engine. |
| Performance  | there is a small performance hit because of the virtual hardware. | Almost no performance hit compared to running the application natively on a host OS.      |
| OS           | Each virtual machine has its own OS.                              | We use the host OS.                                                                       |
| Startup time | takes a few minutes to start the OS and the application.          | it takes a few milliseconds to start the application.                                     |
| Storage      | Lots of storage space wasted because of the OS.                   | Container images are small; we only have the application and its dependencies.            |
| Isolation    | Fully isolated                                                    | Isolated on the process level so could be less secure.                                    |

I hope you enjoyed this lesson, if you have any questions, please leave a comment.
