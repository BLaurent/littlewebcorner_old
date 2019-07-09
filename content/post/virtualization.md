---
title: "VMs Everywhere"
date: 2018-05-09T12:27:19+02:00
draft: false
image: "img/portfolio/budapest.jpg"
tags:
- kvm
- ESXi
- Xen
categories:
- post
description: "All about VMs"
---

# Few basic concepts

## What is Virtualization?
This concept is not related to Cloud or Software in the beginning, but rather creating a virtual yet real version of something to be seen as the real thing. Expression like *Virtual Reality* means exactly that. 
When applied to hardware, **virtualization** create a complete machine hardware with defined capabilities to be exposed to guest OS.
As result guest OSs run on complete isolation from each other, each one with a predefined hardware. This means that bare metal Hardware is *shared* between guest OSs. Guest OSs are not aware that is actually sharing anything at all. As direct result even if modern CPUs offer specific instruction (Intel VT-x & AMD-V) to accelerate virtualization there is still a performance cost. Before the Virtualization it's was single OS per machine and strong dependencies between software and hardware, underutilized resource (idel time).
While being negligeable or not taken in account for most applications, heavy computational intensive tasks like Deep-Learning tends to require more flexible strategies like vGPU or passthrough.

![Virtualization High Level](/images/virtualization-high-level.png)

## Multiple virtualization solution
The *Virtualization* layer color in the diagram above shows that we replace the OS layer by the virtualization layer or **Hypervisor**. On the market there is several hypervisor that could be used some a open sourced some are proprietary. We can named few of them:

* KVM
* Xen
* VMware vSphere / ESXi 
* RT Hypervisor
* Microsoft Hyper-V
* VMWare
* Virtual Box
* gVisor

### Classification of Hypervisor
Depending of the architecture of the hypervisor we can classify them in two categories:

* Type 1 : Native or bare metal hypervisor
    - Run directly on the host hardware 
    - Replace the Host OS
    - Better Performance and stalability
    - Direct Access to Hardware ressources
    - Less hardware support
* Type 2 : Hosted hypervisor
    - Hosted on the main operating system
    - Run as a software on top of the OS
    - Access to hardware calls are delegated to OS
    - Increase overhead performance
    - Better hardware support

### Protection ring
OSes are designed in layers to control and protect ressource access and integrity.
Each layer offering a gateway to allow processes running on a given layer to access layers below in a controled and secured way. There is a small confusion sometimes between layer define in processor and in the OS. X86 Architecture define 4 level of proctection while Linux use only two of them (0 and 3). Some OS like (OS/2) use also ring 2 for I/O, its really up to OS design to use or not those different rings level.

To summaries x86 process offer the following levels :

* Ring 0 Aka kernel space
    - The lowest level, full access to hardware ressources, this is typically where the kernel is running, 
* Ring 1 & 2 Driver level (in theory because linux kernel only use 0 and 3)
* Ring 3 Application level (user space)
    - This is where applications are running very restricted and secured access to ressources

By default when machine are turn on, the boot loader is loaded then the OS kernel is started boot while still in Ring 0. Once the kernel and various drivers loaded applications can be started in level 3.
Applications intereact with low level ressources like opening a file, allocating memory through system calls. 

All tricks done by hypervisor will present to the guest OSes interface mimicking those ring level while still running in less privileged layer. E.g Virtual Box (Hypervisor type 2) launch guest VM in host ring 1, while still looking at ring 0 from the guest point of view.

### Virtualization technique
Depending on the requirement several methods could be applied:

* Full
   - Abstract hardware layer 
   - Guest OS totaly ignorant that it is a guest
   - OS call Translated on the fly by hypervisor
   - No Hardware Assistance or Modification
* Para
    - OS assisted virtualization
    - Lightweight technique
    - No CPU extension needed
    - Hypervisor exposed an API (hypercalls) and Guest need to use this API => OS modifications (install drivers)
    - Near-Native performance
    - Supported in linux kernel since 2.6.23    
* Hardware-assited
    - Accelerated through specific processor capabilities
    - Like Full but with CPU support alowing to bubble guest OSs at lower level

![Different Approaches to Providing the Virtualization Layer](/images/diff-virt-layer.png)

The level of maturity and the price is very different depending of the solution you picked.
In addition it's important to note that there is some important architectural difference between them. If the following section we will focus only on *KVM*, *Xen*, *ESXi*.

#### KVM
Kvm is splitted in two parts : 

* A Linux kernel used as hypervisor through a kernel modules *kvm.ko*, *kvm-intel.ko* & *kvm-amd*.ko*
* A Device emulator (QEMU) exposing virtual hardware ressources.
- with Virtio addition Paravirtualized is supported for some drivers (Better performance)

![KVM Architecture High Level](/images/kvm-high-level.png)

Combined with Hardware support it's possible to run OSes at Ring 0 within the context of Virtualization.
Hardware support create a nested ring 0 allowing Any OS to run unmodified.

Initiallly KVM didn't support Paravirtualization, forcing things to be runned at ring 0 level, exposing to more potential security issues.

#### Xen
Xen use a different architecture, composed by :

* hypervisor
- as little code a possible to reduce possible issues
* Dom 0 VM privilege, a linux (BSD, Solaris) with a modified kernel to talk to the hypervisor
- Monitor, Start & Stop User Domain VM (Dom U)

Xen proxy all ressources access though Dom 0, creating a defacto security layer segregation between Guest OSes and Host. As for KVM better performance are achieved though paravirtualization, but require installation of specific drivers. 

Having this extra level increase the complexity, and Cloud Provider tend to abandon Xen in favor of KVM nowadays.

![Xen Architecture](/images/xen-architecture.png)


#### ESXi
Built by VmWare it's an evolution of the ESX hypervisor, it's totally standalone solution not leveraging any linux kernel underneath. Ultra light, and very stable, its share a bit the KVM architecture :

![ESXi Architecture High Level](/images/ESXi-high-level.png)

## Advantages and Disavantages

### Advantages

* Better usage of resources:
- One of the major advantage of Virtualization is the consolidation of Hardware resources, less idle time per machine, since many guest OS run of the same hardware. 
* Simplified administration
- Everything is centralized
* Green technology
- Less hardware
- Energy and power saving
* Redudancy & Fault tolerance through culstering
* Simplifed Deployement and management
* Better usage of allocated hardware
- Usage statistic shows improvement from 15% to up to 80%
* Scalability: adding more resources from host to guest
    - Additional processing power
    - Network bandwidth
    - Storage capacity
* High Availabity
   - Allocate more VMs when needed
   - back-up running VMS by snapshoting current state
* Release & deployment of new software release
   - "*Just*" need to regenerate the VM image and spin new version


### Disavantages

   * By nature running Mulitple Guest OS put more pressure on Bare metal hardware
   - More CPU
   - More Ram
   - More Disk IOs
   - More Network IOs

## Who is using virtualization 

* Amazon Eleactic Compute Cloud aka EC2
- Use to run everything on Xen 
- Moved to a customized version KVM since October 2017
* Azure
- Proprietary Solution Hyper-V
* Google Compute Engine (GCE)
- Use KVM but replaced QEMU with homemade solution for security reasons
* Digital Ocean
- Run KVM
* Virtual Box

## Perspective on Virtualization

With the advent of container like docker, things have change quite a lot. We moved from using many tools (vagrant ansible, packer) to manage and generate VM images to Dockerfile, but overhead is not negligeable for high throughput serivices and kernel bugs are can be harmfull. Because remember running containers still require running VMS underneath. Several innitiative are looking at a new approach removing the need of strong VM isolation using level 2 hypervisor. One solution recently open sourced by Google is named gVisor.

The "old" model virtualizaing using Vms, Hypervisor + Guest OS + Applications/Container 
![](/images/vm-based-container-technology-gvisor.png)


The new approach as persented by gVisor, we keep the Hypervisor we put the isolation layer on top then Applications/Containers.
![](/images/gvisor-sandbox-containers.png)

From a cloud provider point of view offering a container managed service using this technology allow application to run on a very thin layer, without the potential exposure to kernel 0day vulnerabilities.
Let's pause for now on this subject and save for more in a futur arcticle.


# References

* https://cloudplatform.googleblog.com/2018/05/Open-sourcing-gVisor-a-sandboxed-container-runtime.html
* P. Yu et al., "Real-time Enhancement for Xen Hypervisor," 2010 IEEE/IFIP International Conference on Embedded and Ubiquitous Computing, Hong Kong, 2010, pp. 23-30.
doi: 10.1109/EUC.2010.14
keywords: {operating systems (computers);real-time systems;scheduling;virtualisation;Xen hypervisor;Xen scheduler;Xen virtualization platform;embedded systems;real-time operating system;server consolidation;system virtualization;Credit scheduler;Real-time virtualization;Xen;hypervisor},
URL: http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5703494&isnumber=5703490 
* Fayyad, Hasan & , LucPerneel & Timmerman, Martin. (2013). Benchmarking the Performance of Microsoft Hyper-V server, VMware ESXi and Xen Hypervisors. Journal of Emerging Trends in Computing and Information Sciences. Vol. 4, No. 12 , December 2013, pp: 922-933, ISSN 2079-8407. 