---
layout: post
title: "Openstack and Cloudera workshops at Velocity"
date: 2011-06-28 21:47
comments: true
categories: 
- velocityconf
- Cloudera
- Hadoop
- OpenStack
- IaaS
---

The first workshop I attended at Velocity was run by Rib Pedde, Todd Willey, and Matt Ray, and discussed the Openstack project. This was less of a workshop, and more of a talk in three parts. Interesting, none-the-less. Openstack is a virtualisation platform which has gained quite a lot of momentum in the last year.

http://velocityconf.com/velocity2011/public/schedule/detail/19970

The majority of the session provided an overview of the different components of OpenStack as they stand today and a look at the project's road map. It seems that a lot has happened to this project over the last couple of months and they seem to have quite an interesting set of features coming in the next couple of months.
<!-- more -->

As the speakers presented it, there are two core components Openstack: OpenStack Compute (or Nova), and OpenStack Object Store, (or Swift). In Amazon speak, these are the OpenStack equivalents of EC2 and S3. Both the virtualisation and storage layers aim to be scalable, back-end agnostic, and use eventually consistent.

The speakers went through the architecture of a what a compute cluster would look like. Compute nodes run the virtual machines and are typically commodity servers with a public, private, and management network. After the talk, some people were asking whether all three interfaces were necessary. The speakers said that they weren't, but they did recommended separating management/private traffic from the public traffic.

One of the interesting features was to see they've got an EC2 compatible which is nice. Another thing that was interesting is that while they store metadata in SqlAlchemy for fast access, the canonical source of the cluster's state is derived from the state of the virtualisation layer. Not sure exactly how they're deriving firewall and networking configuration - that's one for me to read more about.

Another feature I liked was the concept Utility VM - a kind of virtual equivalent of a hardware appliance. A Utility VM essentially allows them to do various virtualisation offerings such as Database as service.

They also discussed their API, which unsurprisingly makes use of async rest calls. Someone asked if they had plans to provide a synchronous api, the response was that clients should get feedback by polling. However, the speakers indicated that there might be support for a pub-sub mechanism with callbacks at some point in the future. I found the fact they were considering callbacks interesting. Although it would add state to their infrastructure, it's clearly a feature that clients want.

Interesting upcoming features:

* Quantum: Networking as a service (with Cisco)
* Burrow: HTTP message queue
* Red Dwarf: Database as a service
* Keystone - pluggable auth for all OpenStack components
* lunr - volumes as service
* A dashboard to manage the stack
* Atlas: load balancing as a service (backed by Zeus)

All of these components are clearly being designed with a specific initial vendor in mind (e.g. Zeus for their load-balancing solution) but the general OpenStack philosophy seems to be to make these back-ends pluggable and introduce additional back-ends later on.

They then talked a little around the design of Swift, and how enterprise storage was expensive and requirements were going to double every year. Swift needed to be designed for failure with self healing using commodity hardware. To provide resilience, Swift has the concept of an 'availability zone' inside a cluster... it's an overloaded term, but an OpenStack avaliablity zone they mean a set of storage resources that might all die together in the event of a network, cooling or power failure. Data is then stored in 3 different zones.

A other few parts of the architecture I found interesting:

* To manage where the data is stored they have a coordination function named the Ring. Interesting story of how the Ring's implementation evolved. They started with a DB and although they knew it wasn't going to scale long term it quickly became unmanageble. It was an interesting lesson-learnt that they "didn't realise how amazingly quickly [a relational db] becomes a bottleneck". One member of the audience asked if the ring was the bottleneck... the speakers thought that networking would be the true bottleneck.
* To achieve better scalability they distributed the work by pushing work into the replicas. They defer checking and auditing etc to the replicas rather than in a central coordination, and they now store metadata in sqllite databases within Swift.
* Objects are versioned using system time, so at Rackspace everything's synchronised over NTP. For the distributed databases (sqllite stored in Swift), the NTP timestamps are used for row-based synchronisation

Finally, Matt Ray talked a little about using Chef to manage OpenStack. He gave an intro to Chef and the idea of infrastructure as code.

Knife openstack is a project to bring the equivalent management utilities to ec-tools.
The other interesting tool he mentioned was Crowbar from Dell which is a PXE-based tool to bootstrap openstack and chef.
One interesting point he made was that OpenStack was evolving so fast that he was having to create a separate Chef cookbook for each major version. I found that interesting because we've seen a similar issue with needing to manage and version several parallel branches of Puppet config.

The final workshop I attended at Velocity was

"Managing the System Lifecycle and Configuration of Apache Hadoop and Other Distributed Systems"
/ Philip Zeyliger Cloudera

The talk introduced Hadoop and it's various components, which was handy as there always seem to be so many.

* HDFS
* MapReduce
* ZooKeeper
* HBase
* Oozie
* HUE

He discussed how different people were managing Hadoop hosts (PXE, kickstart, Cobbler) and software (packaging, file transfer, installer scripts).

An interesting comment made during the talk were that we still talk about distributed systems are clusters of nodes whereas Hadoop is all about programming a datacentre as if it were one node. So, for instance, puppet and chef are designed to manage N hosts running a number of services... but with hadoop you're trying to manage a N hosts running ONE distributed/failsafe service.

He introduced the Cloudera Service and Configuration Manager (SCM) which is essentially a central console with log and status aggregation and the Cloudera Activity Monitor, which he described as being like UNIX top for hadoop. 