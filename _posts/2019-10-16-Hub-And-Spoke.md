---
layout: post
title:  "Hub and spoke architecture with NiFi Site-To-Site at any level (a NiFi 1.10 series)"
author: abdelkrim
categories: [NiFi, NiFi-1.10]
image: assets/images/3-1.png
description: "Hub and spoke architecture with NiFi Site-To-Site at any level (a NiFi 1.10 series)"
toc: true
comments: true
---
The Apache NiFi community did an amazing job working on improving NiFi and adding new features. Apache NiFi 1.10 comes with 360+ Jira closed with big improvements/new features sections. Numerous bugs have been resolved too, which makes NiFi more robust.

In this blog, we will see how NiFi 1.10 makes it easier to use, secure, and version flows that use Site-To-Site (S2S) communications.

## What’s NiFi Site-To-Site?

Before jumping into the changes that the new NiFi brings to S2S, let review some concepts and use cases. S2S is an Apache NiFi internal protocol used to make several NiFi instances talk in a bidirectional way. It also used by MiNiFi agents to exchange data with a NiFi instance. S2S makes it possible to build multi-region multi-tier distributed architectures for several architectures. Let’s take a look at two use cases.

### Internet of things

In IoT use cases, MiNiFi agents are deployed on hundreds or thousands restricted edge devices to collect machine data and send it back to the Cloud or the data center. These edge devices are deployed in a large geographical distributed zones that can spans several cities/countries. Connected cars are a typical example of this use case. In these settings, it’s not efficient to send data from the edge to the central platform because of latency, bandwidth availability or regulation considerations. An intermediate NiFi cluster, called Branch cluster, can be deployed to collect data from regional MiNiFi agents and send the data back to a central NiFi cluster through a a more secure and efficient network. The architecture looks like the below picture at a high level.

### Cloud/On-prem data migration

NiFi has several cloud connectors to read/write data from object stores, cloud databases, brokers etc. A NiFi instance deployed on prem can read data from an Oracle database and send it directly to BigQuery and S3 for Cloud use cases. This works well when you have an open network to all the destinations you need to reach and when the connector’s native security is at acceptable level for you security team. In real-life, the situation get more complex as you need to open particular ports for each new data flow you need to deploy. It’s even harder if you need to bring data from the Cloud to the on-prem because It’s not rare that inbound connections are restricted or forbidden. Finally, for performance consideration, it’s often advised to have local producers to not hit timeouts and network disconnections for every data transfer. For all these reasons, it may make sens to have two NiFi instances, one on-prem and one on the Cloud, talking together through S2S. Each NiFi instance is responsible for local read and write from it’s close data sources, and S2S is responsible for moving data through the internet in a secure and efficient way. Indeed, S2S comes with native two way authentication with certificates, support for high availability and load balancing between NiFi nodes, and smart event batching for efficient data transport.

### Data Load Balancing

Before NiFi 1.8, S2S were also used load balancing data between NiFi nodes especially when implementing a List/Fetch pattern. To do so, A NiFi cluster would send data to itself through S2S triggering a redistribution of data across cluster nodes. Note that this use case is no more a valid one because NiFi 1.8 introduced Load Balancing for relationships which easier to use. If you are still using S2S to rebalance data accros your NiFi cluster you really need to read [the following article](https://pierrevillard.com/2018/10/29/nifi-1-8-revolutionizing-the-list-fetch-pattern-and-more/).

## S2S, Input Ports and Remote Process Groups

S2S communications in NiFi are based on two components: Input Ports and Remote Process Groups (RPG). To send data from a NiFi Instance A to a NiFi instance B, we need to add an RPG in NiFi A configured with the address of NiFi B. We also need to add an Input Port to NiFi B to accept incoming data. It’s also possible to use Output Ports for pull use cases with the same logic.

![photo]({{site.baseurl}}/assets/images/3-2.png)

For this to work, the S2S protocol forces you to have the input port defined at the root level of the canvas. By the root level, I mean it cannot be contained within a process group. You can notice this in the left-low corner of the on-prem NiFi UI. In a multi-tenant environment this creates two issues.

1- For flow visibility and versioning, it is recommended to organize NiFi flows into different Process Groups. This creates a multi-tenant organization where you can fine tune ACLs and easily version/push/pull flow developments. Having a hierarchy of Process Groups is something common in real world deployment. An example of such hierarchy can be “Business Unit” -> “Department” -> “Project” -> “Team” -> “Data Source” -> ****. So what’s the impact of this organization on using S2S? well, since S2S requires an Input Port at the root level, we have to use a succession of Input Port from the root canvas to the Process Group where we are actually processing the data.

![photo]({{site.baseurl}}/assets/images/3-3.png)

2- In a secure multi-tenant cluster, users have read/write rights only inside their process groups. When a new project is on-boarded, the NiFi admin creates the process group and changes its permissions to give the right privileges to the designers. This allows for a better separation between roles and makes each team autonomous. However, to use S2S, an input port need to be created at the root canvas which is out of the scope of our designer. They need to interact with the NiFi admin each times to create the Input Port and all the downstream relations until reaching a level that the designer can control. Cumbersome!

## What NiFi 1.10 brings to the table?

Thanks to NIFI-2933, NiFi 1.10 treats Remote input/output ports and local process groups as completely different components which makes it possible to receive data through S2S at any level! When a designer adds an Input Port inside a process group, NiFi asks explicitly if this port is local or should be used for a S2S communication with the external world.

{:refdef: style="text-align: center;"}
![photo]({{site.baseurl}}/assets/images/3-4.png){: .center-image }
{: refdef}

By enabling this granular configuration, an Input Port created inside a process group as a Remote Connections will be visible from the outside world. And then, available to be used in a S2S communication as you can see in the below two pictures (note the PG hierarchy in the left-low corner of the on-prem NiFi UI)

![photo]({{site.baseurl}}/assets/images/3-5.png)
{:refdef: style="text-align: center;"}
![photo]({{site.baseurl}}/assets/images/3-6.png){: .center-image }
{: refdef}

## Conclusion

Site-To-Site communication is a great feature that supports data exchange between several NiFi/MiNiFi instances in a secure and efficient way. It’s the basis of several use case such as IoT and Cloud/Onprem data migration. Previous NiFi versions forces you to have the input port defined at the root level which makes multi-tenancy and security more complex. The latest release of Apache NiFi 1.10 unlock the potential of S2S by accepting Remote Connection from anywhere inside a NiFi hierarchy. In the next blog, I’ll cover new features for data enrichment.

Thanks for reading me. As always, feedback and suggestions are welcome.
