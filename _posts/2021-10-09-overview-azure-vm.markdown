---
layout: post
title: Overview of Azure Virtual Machines
date: 2021-10-09 15:30:00 +0700
description: Overview of Azure Virtual Machines. # Add post description (optional)
img: card-1280.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Azure, Virtual Machines, IaaS]
---
[Azure Virtual Machines](https://azure.microsoft.com/en-us/services/virtual-machines/) is the infrastructure-as-a-service offering in [Microsoft Azure](https://azure.microsoft.com/en-us/). This post provides a brisk overview of Azure Virtual Machines focused on the following topics:
-	Compute
-	Disks
-	Network
-	Agents / Extensions
-	Monitoring
-	BC/DR
-	Host Maintenance

Azure Virtual Machines deploys Windows or Linux servers as Hyper-V guest VMs. When deployed each VM typically has:
-	CPU
-	RAM
-	OS disk durably persisted in Azure Storage
-	temporary disk attached to the physical host
-	zero or more data disks durably persisted in Azure Storage
-	one or more virtual NICs

The post will discuss these topics in more detail.

There are many ways to place a VM in Azure during deployment and these are considered in an earlier [post](https://soitgoes.dev/vm-placement/).

# Compute

## VM Families
Azure Virtual Machines provides a [wide variety](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes) of compute families, each focused on specific compute needs through different combinations of CPU and RAM ranging from 1 vCPU and 0.5 GiB RAM up to 416 vCPU and 5,700 GiB RAM. These sizes are grouped into families targeted at specific workloads by having similar CPU capabilities and vCPU/RAM ratios. Mainstream SKUs are available in all Azure regions while specialty SKUs, such as GPU SKUs, are available in many Azure regions. There may be multiple generations, or series, of each family – the D-series family currently has v5 in preview, for example. The names of VM families, series, and SKUs roughly adhere to a documented [naming convention](https://docs.microsoft.com/en-us/azure/virtual-machines/vm-naming-conventions).

### Mainstream
The D, E and F families provide the mainstream Azure VM SKU. The Azure documentation refers to them as general purpose, memory-optimized, and compute optimized respectively. However, it is easier to view them as the low memory, standard memory, and high memory mainstream SKUs since F has 2GiB per vCPU, D has 4GiB per vCPU, and E has 8 GiB per vCPU. For a given generation, the D and E families have the same CPU and are identical other than the RAM/vCPU ratio.

The D v4 and E v4 generations come in three distinct series, one with an AMD CPU and two with an Intel CPU. The [Dav4/Dasv4](https://docs.microsoft.com/en-us/azure/virtual-machines/dav4-dasv4-series) and [Eav4/Easv4](https://docs.microsoft.com/en-us/azure/virtual-machines/eav4-easv4-series) series use the AMD EPYC™ 7452 while the [Ddv4/Ddsv4](https://docs.microsoft.com/en-us/azure/virtual-machines/ddv4-ddsv4-series) and the [Edv4/Edsv4](https://docs.microsoft.com/en-us/azure/virtual-machines/edv4-edsv4-series) series use the Intel® Xeon® Platinum 8272CL. The Intel variant has higher performance for cached throughput and IOPS – and is priced higher than the AMD series. The second intel series is the [Dv4/Dsv4](https://docs.microsoft.com/en-us/azure/virtual-machines/dv4-dsv4-series) and [Ev4/Esv4](https://docs.microsoft.com/en-us/azure/virtual-machines/ev4-esv4-series) and is distinguished by the absence of a local temporary disk, which means it doesn’t support disk caching – but is otherwise identical. 

### Storage Optimized
The storage optimized [Lsv2](https://docs.microsoft.com/en-us/azure/virtual-machines/msv2-mdsv2-series) series uses locally attached NVMe disks to provide [extremely high disk IOPS and throughput](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/storage-performance). One 1.92TB NVMe disk is attached to the VM for every 8vCPUs it has. The largest SKU, L80s_v2, can hit 3.8M IOPS and 20,000 MB/s across all the NVMe disks. The NVMe disks are directly attached to the physical host, so any data on them will be lost if the VM is deallocated or moved to a new host during a server-healing maintenance operation. The primary focus of this SKU series is distributed databases, like MongoDB and Cassandra, where the database replicates data across multiple instances – mitigating this data-loss problem. While it is possible to attach durable data disks persisted in Azure Storage to Lsv2 VMs this is not the intended use case.

### Memory Optimized
The [Msv2 and Mdsv2](https://docs.microsoft.com/en-us/azure/virtual-machines/msv2-mdsv2-series) series provide very high CPU and RAM options making them good for workloads like SAP which require powerful servers. The two series differ only in that the Msv2 series does not have a local disk so does not support disk caching. The largest VMs in these series, the M192ims_v2 and M192idms_v2, provide 192 vCPUs and 4,096 GiB RAM.

### Compute Optimized
The [FX](https://docs.microsoft.com/en-us/azure/virtual-machines/fx-series) family provides a high vCPU clock speed (4.0GHz Turbo) and up to 1,008 GiB RAM to support workloads requiring high single-core performance.

### HPC
The [HC](https://docs.microsoft.com/en-us/azure/virtual-machines/hc-series) series has a single SKU, HC44rs, which has 44 vCPUs (not hyper-threaded) and 352 GiB RAM. It also exposes a 100Gb/s Mellanox EDR InfiniBand providing low microsecond latency between VMs. The HC series is ideal for workloads requiring intense computational mathematics.

The [HBv3](https://docs.microsoft.com/en-us/azure/virtual-machines/hbv3-series) series has a SKU with 120 AMD EPYC™ 7003 vCPUs (without hyper-threading) and 448 GiB RAM - as well as constrained SKUs with the same RAM and disk configuration but fewer vCPUs. They support 350GBps memory bandwidth and a 200Gbps InfiniBand network allowing them to support supercomputer-scale MPI workloads. The HBv3 series comes with 2 locally-attached [960 GB NVMe disks](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/hpc/hbv3-series-overview#temporary-storage) supporting up to 7GBps reads and 201,000 IOPS for writes.

### GPU
Azure provides three distinct families of VMs containing GPUs: NC optimized for compute-intensive, GPU-accelerated applications; ND optimized for scale-up and scale-out deep learning and HPC applications; and NV optimized for remote visualization. Current versions of these use NVIDIA or AMD GPUs. It is essential when selecting which series to use to ensure it is appropriate for the specific workload and that any needed graphics drivers are available.

- [NCv3](https://docs.microsoft.com/en-us/azure/virtual-machines/ncv3-series) series provides up to 4 NVIDIA Tesla V100 GPUs 

- [NCT4v3](https://docs.microsoft.com/en-us/azure/virtual-machines/nct4-v3-series) series provides up to 4 NVIDIA Tesla T4 GPUs.

- [ND v4](https://docs.microsoft.com/en-us/azure/virtual-machines/nda100-v4-series) series has a single SKU, the ND96asr_v4, which has 8 NVIDIA A100 Tensor Core GPUs. Each VM comes with a 200 Gbps NVIDIA Mellanox HDR InfiniBand connection which allows up to 1.6Tbps of GPUDirect RDMA.

- [NVv3](https://docs.microsoft.com/en-us/azure/virtual-machines/nvv3-series) series provides up to 4 NVIDIA Tesla M60 GPUs.

- [NVv4](https://docs.microsoft.com/en-us/azure/virtual-machines/nvv4-series) series provides fractional GPUs with from ¼ to 1 AMD Radeon Instinct M125 GPU in each VM.

### Burstable
The [B](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes-b-series-burstable) series provides a cost-effective way to host workloads which only occasionally require more power. They have a baseline performance during which credits are built up which can then be burnt down during periods when the VM requires higher performance.

### Constrained
Some high-memory series support the concept of [constrained vCPU](https://docs.microsoft.com/en-us/azure/virtual-machines/constrained-vcpu), which means that only one quarter or one half of the configured vCPUs are exposed to the VM. The intent of these series is to provide the high memory of the configured VM but with a reduced vCPU count to minimize licensing costs for any software licensed by core count which is to be installed on the VM. For example, the E64-16s_v4 is identical to the E64s_v4 except that it has only 16 vCPUs instead of 64 vCPUs. The Azure price is the same in both cases. 

## Images
An Azure VM is typically [created](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal) from an image. There is a wide variety of OS images in the Azure Marketplace ranging from Windows Server 2012 Datacenter to Ubuntu Server 2021.04. There are also many special-purpose images from partners allowing for the creation of specialized servers like network virtual appliances from Firewall vendors. Each image is identified by a combination of publisher, offer, SKU, and version – a specific example being:

- Publisher: Canonical
- Offer: 0001-com-ubuntu-confidential-vm-focal
- SKU: 20_04-lts-gen2
- Version: 20.04.202109080

There are many other Canonical images with different offers, SKUs and Versions.
Azure provides a variety of ways to create images. The easiest way is to create a VM, configure it as desired, create a [generalized image]()https://docs.microsoft.com/en-us/azure/virtual-machines/windows/capture-image-resource using the appropriate tooling, and then create an Azure managed image resource from this generalized image. It is also possible to create an Azure managed image resource from a specialized image. The difference between a generalized image and a specialized image is that all identity information has been stripped out of a generalized image. This makes generalized images a better source for large scale deployments, since there remains the possibility of name collisions when a specialized image is used to create more than one VM in the same virtual network. [HashiCorp Packer](https://www.packer.io/docs/builders/azure) can also be used to create a generalized image out of a configuration file. [Azure Image Builder](https://docs.microsoft.com/en-us/azure/virtual-machines/image-builder-overview) provides this as a managed service.

A VM can be created from a managed image resource – and a single managed image resource can be used to create an unlimited number of VMs.  A managed image resource is very similar to a managed disk resource and can be copied inside Azure or imported from locations outside Azure. The [Azure Shared Image Gallery](https://docs.microsoft.com/en-us/azure/virtual-machines/shared-image-galleries) is a service which simplifies the use of managed image resources and especially their transfer to other regions, since a VM can only be built from a managed image resource hosted in the region which will host the VM.

## Additional Azure Virtual Machine Features

### Generation 1 and Generation 2 VMs
Azure has supported Generation 1 Hyper-V VMs since the service was released.  Azure now supports [Generation 2](https://docs.microsoft.com/en-us/azure/virtual-machines/generation-2) Hyper-V VMs as an option for most VM series and as mandatory for some.  [Gen2 VMs](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v) use UEFI boot architecture instead of the BIOS boot architecture of Gen 1 VMs. Gen 2 VMs also allow OS disks larger than 2 TiB. Not all Hyper-V Gen 2 features are supported in Azure. However, Gen 2 has preview support for [Trusted Launch](https://docs.microsoft.com/en-us/azure/virtual-machines/trusted-launch) for Azure VMs which improves the security of these VMs.

### Nested virtualization
Some [Azure VM families](https://azure.microsoft.com/en-us/blog/nested-virtualization-in-azure/) support [nested virtualization](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/nested-virtualization), which allows an Azure VM to be used as a host for the creation of guest VMs.

## Cost
An Azure VM is billed for every minute it is deployed. The [price](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/windows/) of a VM depends on the SKU and the region - and inside a series the price typically scales linearly with thevCPU count. There is a charge for the VM itself and there may be additional charges for licenses for Windows, or other Microsoft or 3rd-party software installed on it. The durable disks attached to the VM are also billable, and they remain billable while they exist regardless of the deployment status of the VM.
Azure provides a variety of mechanisms to reduce the effective price of Azure VMs.

- Spot Instances
- Reserved Instances
- Azure Hybrid Benefit

### Spot Instances
Many VM families can be deployed as [Spot Instances](https://docs.microsoft.com/en-us/azure/virtual-machines/spot-vms) in which a customer can offer a maximum price for a VM, and it will be deployed while the current spot price for that SKU is lower than the offered priced. The VM will be evicted automatically when the spot price goes above the offered price. Spot pricing provides a useful way to get a lower effective price for workloads which can survive occasional eviction – such as high-scale batch jobs or dev/test workloads.

### Reserved Instances
Many VM families offer one- or three-year reservations at significant discount. These [Reserved Instances](https://azure.microsoft.com/en-us/pricing/reserved-vm-instances/) require pre-payment for the specified term but the VMs are thereafter not billed. The reservation is made at some scope (e.g., subscription) and for some VM series grouping or SKU, and region. Any VM meeting the specified reservation can benefit from the reservation – it is not possible to associate specific VMs with the reserved instance benefits. Azure supports the ability to trade in a reservation for a new reservation 

### Azure Hybrid Benefit
Azure Hybrid Benefit provides discounts for VMs where there is also an existing on-premises license. The [Azure Hybrid Benefit for Windows Server](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/hybrid-use-benefit-licensing) allows customers with Software Assurance to use on-premises Windows Server licenses in Azure, thereby reducing the price of Azure VMs. The [Azure Hybrid Benefit for Linux VMs](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/azure-hybrid-benefit-linux) allows customers to convert their existing on-premises Red Hat Enterprises Linux and SUSE Linux Enterprise licenses to cover the equivalent license cost for RHEL and SUSE VMs running in Azure.

# Azure Disks
A typical Azure VM comes with an OS disk durably persisted in Azure Storage, a temporary disk attached to the physical host, and zero or more [data disks](https://docs.microsoft.com/en-us/azure/virtual-machines/managed-disks-overview) durably persisted in Azure Storage. This configuration ensures that critical data on the OS and data disks will survive the deallocation of the VM although everything on the temporary disk will be lost. The number of data disks which may be attached to a VM depends on the VM size.

## Disk Types
Azure provides the following durable [disk types](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types) where the disks are durably persisted in Azure Storage:

- [Premium SSD](https://docs.microsoft.com/en-us/azure/virtual-machines/premium-storage-performance)
- [Standard HDD](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types#standard-hdd)
- [Standard SSD](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types#standard-ssd)
- [Ultra Disk](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-enable-ultra-ssd?tabs=azure-portal)

Premium SSD has low-millisecond latency on SSD disks. For disks 512 GiB and smaller, Premium SSD supports credit-based bursting at no cost which temporarily boosts performance up to 3,500 IOPS and 170 MB/s. 1TiB and larger disks can optionally be configured to support on-demand bursting (preview) which can increase performance up to 30,000 IOPS and 1,000 MB/s for the largest size.

Standard HDD has latency in the 10ms range. Standard HDD provides up to 500 IOPS and 60 MB/s for disks 4 TiB and smaller with performance increasing to 2,000 IOPS and 5000 MB/s for the largest disks.

Standard SSD has the latency characteristics of Premium SSD but the throughput and IOPS characteristics of Standard HDD. For disks 1 TiB and smaller, Standard SSD supports credit-based bursting at no cost which temporarily boosts performance up to 1,000 IOPS and 150 MB/s depending on the size.

Ultra Disks have sub-millisecond latency along with provisioned throughput and IOPS which can be configured in ranges depending on the size of the disk. For 1 TiB and larger disks performance can be configured up to 160,000 IOPS and 2,000 MB/s. Provisioned throughput and IOPS can be changed at any time.

Premium SSD is recommended for all disks in production VMs. If the highest provisioned throughput or IOPS is needed Ultra Disks should be used for data disks.

## Disk Performance
The disk performance exposed into a VM depends on the disk throttling targets of the VM and the attached disks. For most VM sizes Azure supports disk caching, and when configured for data disks this typically increases the upper limit for IOPS but lowers the upper limit for throughput. It is important that the VM size and disks be balanced to ensure that the desired performance is achievable. For example, a small VM will throttle disk traffic well below the documented throttling targets for a large Premium SSD disk. Note that Azure disks are optimized for parallel access so reaching the highest performance requires ensuring a high enough queue depth.

## ZRS
Azure managed disks are configured to use LRS storage which makes 3 synchronous replicas of the data in the host region. There is a limited preview of a [ZRS storage](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-deploy-zrs?tabs=portal) option which will guarantee that the 3 replicas are distributed across 3 availability zones in regions with availability zones. This feature increases the reliability of the application on the VM since if the zone hosting the VM goes down a new VM can be built in a functional zone using the existing disks in a different zone.

## Disk Arrays
Azure VMs support the use of disk arrays to increase the effective size of the disks exposed into the VM. On Windows Servers, [Storage Spaces](https://docs.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/storage-configuration?tabs=windows2016) can be used to create a disk array. On Linux servers, MDADM or [LVM](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/how-to-configure-lvm-raid-on-crypt) can be used to create a disk array. Note that disk arrays are typically deployed using RAID-0 because Azure Storage already stores 3 synchronous replicas of all the data in each disk.

## Shared Disks
[Azure Shared Disks](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-shared) provides the ability to share a managed disk among several VMs. This disk can be of any type except Standard-HDD. VMs using Shared Disks must be configured to use a cluster manager like Windows Server Failover Cluster or Pacemaker. This feature is primarily intended to support the migration of workloads which had been using Storage Area Networks (SANs).

## Locally-Attached Disks
A typical Azure VM has an OS disk stored in durable Azure Storage and a temporary disk attached to the physical host. Azure supports some variants of this model. With an [Ephemeral OS disk](https://docs.microsoft.com/en-us/azure/virtual-machines/ephemeral-os-disks) attached to the physical host an Azure VM can have all dependencies on Azure Storage removed. A downside is that the VM is deleted when deallocated since the ephemeral OS disk is lost. Some VM series have no temporary disk – for example, the Dv4/Dsv4 and Ev4/Esv4 – so do not support disk caching. The [Lsv2](https://docs.microsoft.com/en-us/azure/virtual-machines/lsv2-series) series has an OS disk hosted in durable Azure Storage, a temporary disk attached to the physical host, and one 1.92TB NVMe disk attached to the physical host for each 8 vCPUs. The L80s_v2 has 19.2TB of locally attached NVMe disks providing 3.8M IOPS and 20,000 MBps of throughput. Note that the content of these NVMe disks is lost if the VM is deallocated or moved to a new host through a server-healing maintenance operation.

## Disk Encryption
Azure provides [several](https://docs.microsoft.com/en-us/azure/virtual-machines/disk-encryption-overview) disk encryption features:

- [Server-Side encryption](https://docs.microsoft.com/en-us/azure/virtual-machines/disk-encryption#encryption-at-host---end-to-end-encryption-for-your-vm-data) – disks durably persisted in Azure Storage have the data automatically encrypted using either platform-managed or customer-managed keys.
- [Host Encryption](https://docs.microsoft.com/en-us/azure/virtual-machines/disk-encryption#encryption-at-host---end-to-end-encryption-for-your-vm-data) – the ephemeral OS disk and temporary disk attached to an Azure VM are optionally encrypted by the physical host using platform-managed keys
- Disk Encryption – the disks on the VM are encrypted using either [BitLocker](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/disk-encryption-overview) or [DM-Crypt](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/disk-encryption-overview) using keys stored in Azure Key Vault.

# Network
## NICs
An Azure VM typically has a single virtual [network interface card (NIC)](https://docs.microsoft.com/en-us/azure/virtual-network/network-overview#network-interfaces) attached to it. This NIC typically has a single static private IP address in a subnet in an Azure virtual network in the same subscription and region as the VM. The NIC can also be configured to have a static public IP address on the Internet. A Network Security Group can be used to restrict ingress and egress traffic through the private and public IP addresses.

The general configuration is that an Azure VM can have one or more NICs, each of which can have one or more private IP and public IP addresses, each of which can be either static or dynamic and support either IPv4 or IPv6. In a multiple NIC configuration one of the NICs must be configured as the primary and is used as the default route for egress from the VM. Additional configuration is needed on the VM to allow traffic to flow out on the secondary NICs.

The number of NICS and the egress [throughput limits](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-machine-network-throughput) are dictated by the [VM size and family](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes). The upper limits for the latest generation of a VM family are typically 8 NICs and about 30,000 Mbps. Note that the throughput limit is shared among all the NICs, so adding more NICs does not increase the maximum throughput.

Azure uses DHCP for the private IP addresses of all VMs in a virtual network. Static IP addresses are implemented using a dynamic IP address with a multi-decade lifetime.

The general guidance is to not attach a [public IP address](https://docs.microsoft.com/en-us/azure/virtual-network/ip-services/public-ip-addresses) to a VM and instead use alternative Azure techniques to handle ingress traffic. Even without a public IP address, egress traffic from the VM can still be forwarded to the Internet with Azure using source network translation (SNAT) with an IP address outside customer control. Under certain conditions it is possible for the VM to encounter SNAT port exhaustion, and the caller will encounter network errors. For this reason, the guidance is that egress traffic from the VM should be controlled through use of an Azure NAT Gateway, Azure Firewall, Azure Load Balancer, or a public IP address – with these listed in order of decreasing desirability.

[Accelerated Networking](https://docs.microsoft.com/en-us/azure/virtual-network/create-vm-accelerated-networking-powershell) is an Azure feature where single root I/O virtualization (SR-IOV) is configured for a VM, thereby allowing increased network throughput and reduced latency for TCP and UDP traffic. Accelerated Networking is recommended for any VM which supports it, typically those with four or more vCPUs, and is needed to achieve the documented throughput maximums. Note that accelerated networking does not provide any performance improvement for ICMP traffic so ping statistics cannot be used to identify any performance improvement. Utilities like iperf3 can be used instead to demonstrate the benefit of configuring accelerated networking.

## InfiniBand
Some H-series and N-series VMs can use RDMA over an [InfiniBand](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/hpc/enable-infiniband) network which provides low latency and high throughput over an InfiniBand network. Latencies can be in the low microsecond range, much lower than the close to 1 millisecond latencies typically achieved with standard Azure NICs. The InfiniBand network provides throughput of 200 Gbps but can reach 1.6Tbps on the [ND A100 v4](https://docs.microsoft.com/en-us/azure/virtual-machines/nda100-v4-series) series. Note that InfiniBand requires [special drivers](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/hpc-compute-infiniband-windows) and specific application code so is not used for normal network traffic.

# Agents / Extensions
The [Azure VM Agent](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/agent-windows) is a service which is installed on every Azure VM agent deployed from the Azure Marketplace. The agent provides connectivity between the Azure Fabric and the VM and is used to handle Azure features such as managing Azure extensions installed into the VM and password recovery.

It is highly recommended that the VM Agent be manually installed on every VM deployed in Azure which were not deployed using marketplace images – for example, those migrated from on-premises or created from custom images built outside Azure.

Azure supports a [wide variety](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/overview) of Microsoft and 3rd-party extensions (or agents), which are services deployed into Azure VMs to perform specific tasks. The Azure Monitoring Agent sends log data to Azure Monitor for subsequent analysis. The Custom Script Extension allows PowerShell or bash scripts to be run on demand on a VM without the need to log in to the VM. The [Chef extension](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/chef) allows the VM to be configured using Chef.

# Monitoring
[Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/monitor-reference) is the Azure service enabling the capture, analysis, and visualization of data reflecting the state of an Azure resource. This data can be used to identify the current state of a resource and to trigger mitigation efforts when the resource is not behaving as expected,

Azure Monitor supports the following data:

- Log Analytics – structured logs emitted by Azure resources
- Metrics – time-series database of numeric metrics emitted by Azure resources
- [VM Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/vm/vminsights-overview) – performance, health, and dependency data for a VM

Log Analytics and VM Insights data is stored in a Log Analytics workspace with customer control over the location of the workspace and the longevity of the logs. Metrics data is stored in a Microsoft-managed time-series database. The Kusto Query Language (KQL) can be used to analyze data Log Analytics and Metrics data. Azure Monitor supports visualization of metrics and log analytics data either with the resource in the Azure Portal or in dashboards developed by the customer. The data can also be exported to 3rd-party analytics and visualization services. Azure Monitor supports the creation of alerts based on the data and these alerts can be sent through SMS, phone, push notification

The metrics Azure Monitor captures for the pertinent Azure resources are:

- [Virtual Machines](https://docs.microsoft.com/en-us/azure/azure-monitor/essentials/metrics-supported#microsoftcomputevirtualmachines)
- [Disks](https://docs.microsoft.com/en-us/azure/azure-monitor/essentials/metrics-supported#microsoftcomputedisks)
- [NICs](https://docs.microsoft.com/en-us/azure/azure-monitor/essentials/metrics-supported#microsoftnetworknetworkinterfaces)

Azure VMs also support the use of the [Azure Diagnostics](https://docs.microsoft.com/en-us/azure/azure-monitor/agents/diagnostics-extension-overview) extension which is an agent, installed into the VM, which is used to capture a variety of data including perf counters, ETW traces, and log files. This data can be persisted into Azure Storage and Azure Monitor.

Azure VMs also provides the following health notifications:

- [Service Health](https://docs.microsoft.com/en-us/azure/service-health/service-health-overview) – notification of service incidents which may impact customer resources. 
- [Resource Health](https://docs.microsoft.com/en-us/azure/service-health/resource-health-overview) – notification of an issue with a specific Azure resource for a customer.

Resource Health checks [various](https://docs.microsoft.com/en-us/azure/service-health/resource-health-checks-resource-types#microsoftcomputevirtualmachines) potential issues for a VM when evaluating its health.

[Azure Security Center](https://docs.microsoft.com/en-us/azure/security-center/security-center-introduction) can be configured to monitor some or all VMs in an Azure subscription. It can identify security vulnerabilities on a VM and suggest mitigation for them.

# BC/DR
Azure VMs support two business continuity services: [Azure Backup](https://docs.microsoft.com/en-us/azure/backup/backup-overview) and [Azure Site Recovery](https://docs.microsoft.com/en-us/azure/site-recovery/site-recovery-overview). Both are implemented using a Recovery Services Vault which stores snapshotted copies of the data from the disks attached to the VM. The snapshots are lazily persisted into permanent storage where they can be subject to lifecycle policies such as retention for a specified period.

Azure Backup copies are used to restore the disks for a VM or individual files and folders inside the disk. The vault used to persist backup data must reside in the same region as the VM being backed-up. Note that native database tooling should be used for backing up databases hosted in an Azure VM.

Azure Site Recovery copies are used to restore the disks in a different region so new VMs can be created in that region. The vault used to persist Site Recovery data must reside in the secondary region. Azure Site Recovery provides a simple disaster recovery solution for Azure VMs and is used to failover a VM into the secondary region following an outage in the primary region. 

# Host Maintenance

The physical servers hosting Azure VMs occasionally undergo [maintenance](https://docs.microsoft.com/en-us/azure/virtual-machines/maintenance-and-updates) because:

- host server software is being updated
- host server has failed and the guest VMs have to be moved
- predictive maintenance has indicated a host server failure is likely
- host server is being decommissioned

Most Azure VM families support the use of memory-preserving host updates which pause guest VMs for up to 30 seconds while the host OS update is performed. If the server is failing live migration to a patched physical host can also be used on these VM families, and this causes the VM to pause for up to 5 seconds during the migration.

Occasionally, the host server must be updated in a way that causes it to be rebooted and the VM family does not support live migration. In this scenario, customers are given advance notice and the opportunity to choose the maintenance time or allow it to be done during a platform controlled scheduled-maintenance window. In general, the former is recommended only for business critical individual VMs while the latter is recommended for VMs deployed as part of a group in an availability set or VM Scale Set.

# Summary
This post covered the core features of an Azure VM: compute, storage and connectivity. It also covered areas impacting the reliability of a VM such as: monitoring, backup and connectivity. The post also showed various ways to control the cost of running a VM in Azure.

The official [documentation](https://azure.microsoft.com/en-us/services/virtual-machines/) for Azure Virtual Machines is on the [Azure](https://azure.microsoft.com/en-us/) website.
