---
layout: post
title: VM Placement in Azure
date: 2020-11-23 17:00:00 +0800
description: Overview of VM placement in Azure. # Add post description (optional)
img: azure-blur3.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Azure, Virtual Machines, IaaS]
---
The hosting of VMs is a core Azure service. Azure provides several abstractions constraining the placement of VMs from anywhere in a geographic region to a specific physical host.  This post describes the following placement constraints: 

* Regions
* Availability Zones
* Fault Domains
* Update Domains
* Proximity Placement Groups
* Availability Sets
* Virtual Machine Scale Sets
* Dedicated Hosts
* Isolated Hosts

# Regions

An Azure [region](https://docs.microsoft.com/en-us/azure/virtual-machines/regions) is a collection of one or more datacenters located in a geographical region with the datacenters located within a latency boundary typically several hundred miles from any other region. Azure does not expose the datacenters to customers so there is no way for a customer to know which datacenter inside a region a VM is deployed into.  

A VM must be deployed into a region. If no additional placement constraints are provided, Azure can deploy the VM and its associated disks into any datacenter in the region.

# Availability Zones

An [Availability Zone](https://docs.microsoft.com/en-us/azure/availability-zones/az-overview) comprises one or more datacenters inside an Azure region which are fault isolated from the other availability zones inside the same region. This isolation is sufficient that an outage in one availability zone will not impact availability in the other Availability Zones in the region. There are three or more physical Availability Zones in each region supporting Availability Zones. However, each Azure subscription has only a logical view of these Availability Zones with each subscription having three logical availability zones which are persistently mapped into a set of physical Availability Zones. This mapping is by subscription, so Availability Zone 1 in a subscription is likely to represent a different physical zone from availability zone 1 in a different subscription.

Availability Zones are an important part of the HA story for Azure. Deploying a set of related VMs across all Availability Zones decreases the impact of the failure of a single datacenter or Availability Zone. Consequently, Azure offers a higher [SLA](https://www.azure.cn/en-us/support/sla/virtual-machines) to these VMs and the guidance is to use Availability Zones to get the highest possible availability within a region.

Note that VMs can only be assigned to an Availability Zone at creation time.

# Proximity Placement Groups

A [Proximity Placement Group](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/co-location#proximity-placement-groups) (PPG) is an Azure resource representing a logical container used to constrain a related set of VMs to a single Azure [datacenter](https://azure.microsoft.com/en-us/blog/introducing-proximity-placement-groups/) to minimize the latency between them. The PPG is created in a region and is then pinned to the location of the first VM deployed into it. VMs subsequently added to the PPG will be deployed into the same datacenter as the first one. A PPG resource is deployed into a region but can be associated with a specific Availability Zone by deploying the first VM associated with the PPG into that Availability Zone. A VM can be moved into a PPG after creation provided it is deallocated. Note that the PPG loses its association with a specific datacenter when all the VMs in it are deallocated.

Deploying VMs into a PPG prioritizes latency over availability. There is an increased risk of correlated failure since access to all the VMs can be lost in the event of a failure of the datacenter containing the PPG.

# Availability Sets

An [Availability Set](https://docs.microsoft.com/en-us/azure/virtual-machines/manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy) is an Azure resource representing a logical container used to provide anti-affinity to a related set of VMs. An Availability Set is created in a region or PPG with either two or three fault domains, depending on the region, and from one to twenty update domains with a default of five. When VMs are deployed into the Availability Set they are distributed evenly across the fault domains and update domains.

Different fault domains are isolated from each other to minimize the risk of correlated failure for the VMs in different fault domains. This is often thought of as rack-level isolation – that is, different fault domains would be on different racks.

During platform maintenance operations Azure performs maintenance on the physical hosts hosting all the VMs in one update domain, waits thirty minutes for the VMs to recover from the maintenance, then performs maintenance on the physical hosts hosting all the VMs in the next update domain, and so on.  This allows the VMs deployed into the Availability Set to continue to serve their workload while some percentage of them are being updated.

Note that deployments should be done using managed Availability Sets which aligns the fault domains of the VMs and their attached managed disks. Availability Sets predate the use of managed disks and that alignment is not supported at all for classic unmanaged disks.

Deploying VMs into an Availability Set improves availability over individual VMs. If they are available in the region the use of Availability Zones is preferred over Availability Sets.

There is no direct way to deploy VMs across Availability Zones while deploying them into Availability Sets in each region. This is a common request since it provides the benefit of zone redundancy for availability across the region as well as the anti-affinity of fault domains and update domains inside an Availability Zone. A trick to do this is to create a PPG in each Availability Zone, then create an Availability Set in each PPG, and finally deploy the VMs into the three Availability Sets.

In multi-tier architectures using Availability Sets each tier should be deployed into distinct Availability Sets. There is no relationship between the fault domain and update domains in two different Availability Sets.

# VM Scale Sets

A [Virtual Machine Scale Set](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/overview) (VMSS) is an Azure resource providing a convenient way to deploy a set of identical VMs. These are deployed by specifying a single set of VM properties along with the number of VMSS instances to deploy, with each instance being deployed as a VM. VMSS support auto-scale allowing the deployed number of instances to be modified automatically depending on the performance metrics of the VMSS. It is very common for a VMSS to be deployed behind an Azure Load Balancer or Azure Application Gateway. A VMSS can be deployed into a region, across all Availability Zones, a single Availability Zone, or a [PPG](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/co-location).

The VMSS instances are deployed into [fault domains](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-manage-fault-domains) and update domains. For regions with no Availability Zones there are five fault domains by default. If the VMSS is deployed into Availability Zones, Azure distributes the instances over as many fault domains as possible.

[VMSS Flexible Mode Orchestration](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes#scale-sets-with-flexible-orchestration) is a preview service which facilitates the deployment of a scale set containing either identical VMs or multiple VM types. With Flexible Orchestration the VMs in the scale set can be deployed using the same APIs used to deploy standard VMs.

# Dedicated Hosts

A [Dedicated Host](https://docs.microsoft.com/en-us/azure/virtual-machines/dedicated-hosts) is an Azure resource representing a deployment location mapped to an entire physical host. Customers can deploy one or more VMs into that Dedicated Host so that the VMs are completely isolated from those of other customers and have lower latency between VMs. This isolation may be desired for compliance or regulatory reasons. Dedicated Hosts are supported for a limited set of VM families.

One or more Dedicated Hosts are deployed into a Host Group, which is an Azure resource associated with a region and Availability Zone. To provide increased availability, a Host Group is also associated with some number of fault domains allowing fault isolation for multiple Dedicated Hosts.

Dedicated Hosts provide [maintenance control](https://docs.microsoft.com/en-us/azure/virtual-machines/maintenance-control), which gives customers more control over the timing of platform maintenance.

# Isolated Hosts

Azure supports the deployment of an [Isolated Host](https://docs.microsoft.com/en-us/azure/virtual-machines/isolation) which is a normal Azure VM which takes up an entire physical host allowing customers to have a VM which is completely isolated from those of other customers. This isolation may be desired for compliance or regulatory reasons. Isolated Hosts are supported for a limited set of VM families. An Isolated Host is deployed, and has the same placement constraints, as any other Azure VM – the only difference being that the Isolated Host has a specific size in one of the supported families.

Although a Dedicated Host and an Isolated Host consume an entire physical host the difference between them is that an Isolated Host is a single Azure VM which fits in an entire physical host while a Dedicated Host is used as the deployment location for one or more Azure VMs in the same family.

Isolated Hosts provide [maintenance control](https://docs.microsoft.com/en-us/azure/virtual-machines/maintenance-control), which gives customers more control over the timing of platform maintenance.

# Summary

This post provided a short summary of the various placement options Azure provides for VMs. These allow features including regional or zonal isolation, proximity to reduce latency, anti-affinity for fault isolation and rolling updates, auto-scaling, and deployment into single physical hosts.
