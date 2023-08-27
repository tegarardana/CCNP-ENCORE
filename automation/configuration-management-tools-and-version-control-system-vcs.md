# Configuration Management Tools and Version Control System (VCS)

In this lesson, we’ll take a look at an overview of configuration management tools and version control systems.

## Configuration Management Tools

Servers and network devices don’t remain the same. We install updates for our operating systems and packages and make changes to configurations. Imagine you have to install an update to 10, 100, or 1000 servers or create a new VLAN on 20 switches. These are tasks that you don’t want to do manually, box by box. It’s time-consuming, and it’s easy to make errors if you do everything by hand.

Configuration management tools offer an automated method to implement and monitor changes to our systems. These systems can include servers, storage, networking devices, and software. The goal is to **maintain these systems in known and determined states**. We define the required state in a configuration and the configuration management tool uses automation to match that state on the targeted systems.

Configuration management is important because it allows us to **scale infrastructure and software** without having to scale staff to manage our systems.

### Features

On a high level, configuration management tools offer the following features:

* enforcement
* cooperation
* version control friendly
* change control
* abstraction

Let me explain these features in detail.

#### **Enforcement**

By using a tool to configure our devices, we ensure that the device is configured to the desired state. This **prevents configuration drift**. Configuration drift happens when people manually install packages or change configuration files. **Configuration drift makes troubleshooting time consuming and difficult**.

#### **Cooperation**

Storing configurations in a configuration management tool makes it easier to cooperate with others. Having all configuration files in a single place makes it easy to roll them out to all required systems and to share them.

#### **Version control friendly**

Having all configuration files in a single place means we can put them under version control in a Version Control System (VCS) like Git. This is great for cooperation since multiple people can work together on the same files. Everyone can see who added or edited which files and when.

#### **Change control**

Because configuration management tools are text based, we can put them under version control. With a VCS, we can easily see the changes between configuration files which makes code review simple. This makes it easier to decide which changes to push to the production network.

#### **Abstraction**

Even if you use Linux, it’s possible that you use multiple distributions like Ubuntu and CentOS. There are differences between the distributions. Installing a package requires different commands and the location of configuration files can be different. For example, the Apache webserver uses different files and folders on CentOS or Ubuntu. Configuration management tools abstract these operation system specific items away for you so you can use the same configuration files without worrying about the underlying OS.

### Agent vs Agentless

There are two types of configuration management tools:

* agent based tools
* agentless tools

Agent based tools require the installation of an agent on the system you want to manage. Agentless tools don’t require an agent or software on the system you want to manage.

Puppet and Chef are two examples of agent based tools. Ansible is an agentless tool.

#### **Puppet**

Puppet is a configuration management tool used for deploying, configuring, and managing servers. It uses a master-slave architecture. There is a Puppet master. On the clients we run a Puppet agent that pulls the configuration from the master.

Puppet calls configuration files manifests. Manifests use Puppet‘s own configuration language called puppet DSL.

#### **Chef**

Chef is an automation platform that configures and manages your infrastructure. It uses a master-slave architecture, similar to Puppet. The Chef server manages nodes and stores the configurations for the nodes. Each node runs the chef clients and pulls configuration tasks from the Chef server. Chef calls configurations “cookbooks”.

Users create, test, and maintain configurations or policies on workstations. We push these to the Chef server. Chef uses Ruby DSL for configuration files.

#### **Ansible**

Ansible is a configuration and orchestration tool, written in Python which uses YAML for configuration tasks. We call these tasks playbooks. Ansible is agentless and uses SSH to connect to nodes. It sends programs called Ansible modules, runs them, and removes them when finished.

Ansible is a great way to get started with network automation. Playbooks are easy to read and write and since it’s agentless, you don’t have the hassle of installing a server and agents on your nodes.

You can see some examples of how I use Ansible to manage a Linux server and Cisco switches in the [network automation and orchestration lesson](https://networklessons.com/cisco/ccnp-encor-350-401/network-automation-and-orchestration).

There are many other orchestration and configuration tools out there. [Wikipedia has a good overview](https://en.m.wikipedia.org/wiki/Comparison\_of\_open-source\_configuration\_management\_software) with different tools and their differences.

## Version Control Systems

To understand why we need version control systems, let’s look at an example of what happens when we don’t use one:

_You want to configure an application on a server or configure something on a router or switch. It doesn’t matter what. You want to be smart about it so instead of applying the configuration directly on the server or router,  you create a configuration file and store it on your computer._

_A few days later, one of your colleagues requests your configuration file. You could email the file back and forth but that makes it difficult and you don’t want to share it in plain text so you move the file to a network share so everyone can work on the file._

_A week later, you want to use your configuration file to configure a new router but the router spits out an error._

_Someone changed the file but you have no idea who did it (probably that darned colleague) and what they changed. There is no way to figure it out. You could get a backup and compare the two files but that’s a pain._

Most networks are managed by a group of network engineers. Code projects typically have multiple developers working on the code in parallel. A version control system ensures that there are no code conflicts between developers or errors in configuration files.

Version control systems allow us to keep track of text based files and folders. This has many advantages:

* We can see who made changes to our files.
* It’s easy to create multiple versions of files.
* We can roll back to a previous version.
* It’s easy to work together in parallel.
* It’s easy to share files and folders with others.

There are multiple version control systems like SVN and Git. Nowadays, Git is the most popular choice. Even if you have never heard of or worked with Git before, you have probably run into [github.com](https://github.com/) sometime.

### Git

Git is a free and open source _distributed version control system_. You can use it for small to very large projects. Git was originally developed by Linus Torvalds, the creator of the Linux kernel.

What exactly is a “distributed version control system” ? Let’s break it down:

* **Control system**: Git is a content tracker. This means that we can store content in Git. Git is mostly used to store code but you can use it for any text files, like configuration files for network devices.
* **Version control system**: We add more and more files to Git and these files can change. Git keeps track of all changes.
* **Distributed version control system**: Git uses a local and remote repository. This means that files are not only stored on a local server but all files are also stored on the local computer of a developer.

One powerful feature of Git is branching. Branching allows a developer to “branch out” from the original code and isolate their work from others. This is useful when a developer wants to work on a bug fix or new features. Branching allows each developer to branch out from the original code base and isolate their work from others. It also helps Git to easily merge versions later on.

There are a number of popular web-based options for Git. The three most popular are:

* GitHub
* GitLab
* Bitbucket

These websites allow you to create Git repositories but also have tools so you can easily collaborate with other developers. You can submit issues, leave feedback, etc.

Here is an[ example of one of my Git repositories](https://github.com/networklessons/docker-alpine-freeradius) on GitHub. This Git repository contains the Docker files required to run FreeRADIUS inside an Alpine container. If someone finds an error, they can report the issue. If someone likes the code, they can [fork it](https://github.com/rmcs2002/docker-alpine-freeradius/network/members) and use it for their own work or fix an error, then send a pull request so I can add the fix to my repository.

Explaining Git in detail is outside the scope of this lesson. However, [Github has a great introduction tutorial](https://guides.github.com/activities/hello-world/) to learn the basics of Git. I highly recommend you to give this a try.

## Conclusion

In this lesson, we talked about configuration management tools and version control systems.

The network world is shifting, moving away from the CLI and going towards a world where we use network automation, configuration management, and scripts to manage our infrastructure and network devices. Version control systems like Git used to be “for developers” but nowadays, storing configuration information under version control is just as important.

You learned about the different configuration management tools we can use for our systems and network devices. If this is new to you, I highly recommend trying Ansible. Because it’s agentless, it only takes a few minutes to get started and run your first playbook.

Instead of storing configuration files on your computer or network drives, give GitHub or GitLab a try. Both offer free unlimited private repositories.

I hope you enjoyed this lesson. If you have any questions please leave a comment.
