# Farm-Guide

This is a guide to setup a Phala mining farm, using automation so save you a lot of time. All the steps in these guide and the choices made for e.g. operation system and settings are made for very good reasons. You are highly recommended *NOT* to deviate from these plans. This setup is proven to work, but it also requires basic system operation skills, like updates, docker, ansible, etc. If you are not an expecianced linux administrator this guide will still get you started, but you might face issues you cannot resolve. Please note that this GITHUB is *NOT* a support channel for Phala, if you have questions please use Discord for community support.

## General logic / choices

for miners:
- MAAS installs via PXE an OS on a worker
- Phala-phase1 installs via Ansible all tools, software, drivers that are required
- Phala-phase2 installs via Ansible the docker containers on the worker
- Phala-scripts are some basic tools to use API's and scripted ways to add workers

for services:
- MAAS installs via PXE an OS on a VM
- You install the required software manually (this is a one-time job)
- Follow *all* documentation, dont just skip to tuning. 

# Infrastructre layout

## Routing and IP planning

It does not matter if you have 1 or 10000 miners, you will need a proper network. You will need to make proper planning! If you have 100+ miners you will notice your traffic go up a lot. We measured an 600 miners platform doing 20,3 gbit/s during peak times. In addition you should use layer2 as much as possible, as with many small packets you will notice that 'simple routers' could just die under the heavy load of packets per second(PPS).

It is also recommended to use a good IP plan, again you can make your own, but for this documentation to work you are better for follow it to the letter.

Use IP space 10.xxx.yyy.zzz which add the following logic:
- xxx is the farm location, if you have many.
- yyy is the vlan number subnet
- zzz is the IP usage itself.

Each farm, we call the first 101, must have the following VLAN's:

087 - Service VLAN, this will host your VM's such as full nodes, PRB stack, Docker registry, APT mirrors, etc (10.101.87.0/24)
101 - Worker VLAN, access switch 1 (10.101.101.0/24)
102 - Worker VLAN, access switch 2 (10.101.102.0/24)
...

## Hardware requirements ##

You will need at least 3 servers to act as a VMware (or other type of) Hypervisor with these minimal specs:

-  i5-9600K or a Xeon equilivant
-  64GB memory
-  3TB of NVMe storage
-  3TB of SSD storage (next to the NVMe!)
-  Using VMWare compatible hardware

Using NVMe is a must, the SSD storage is used for 'slower' operations. You can also use 6TB of NVMe is you want to.

## Using MAAS for basic installation of OS ##

We use Ubuntu 18.04 for workers (aka miners) and 20.04 for services. To simplify their installation and management we use MAAS (https://maas.io/)

The installation of MAAS is well documentated on their website, as such not covered in this document ... however you will need to build the following systems:

region 10.101.87.231 / 200GB HDD / 12GB RAM
reqion-sql1 10.101.87.233 / 200GB HDD / 8GB RAM
region-sql2 10.101.87.234 / 200GB HDD / 8GB RAM
rackd1 10.101.87.235 / 40GB HDD / 6GB RAM
rackd2 10.101.87.236 / 40GB HDD / 6GB RAM

if you operate more then 1000 miners, then add 2 (or more) rack controllers for loadbalancing and redundancy, but not required). Always install in pairs for redundancy:

rackd3 10.101.87.237 / 40GB HDD / 6GB RAM
rackd4 10.101.87.238 / 40GB HDD / 6GB RAM

Notice: You should install these on VM's on seperate hypervisors e.g. sql1 on HV1, sql2 on HV2. This is for *redundancy*.
Notice: After installing you must configure the VLAN's and enable PXE on them.
