---
date:  2015-02-06
type: post
title: "How Neutron works"
excerpt: "Neutron internals"
tags: [devstack, multinode, nova, neutron, openstack, vagrant]
---

It's been a long time since I last posted in my blog. Anyway here is one describing
the internals of Neutron with overlay network. You would be able to get most out of this post
if you have a Openstack setup with neutron described in my previous [post](http://). 
There are a number of components which are used in Neutron to give you a software defined
networking.

* OpenVSwitch
* Overlay Network using GRE or VXLAN tunneling
* Linux network namespaces 
* Linux bridges
* Veth pair

### OpenVSwitch

It's a soft virtual switch which supports Openflow rules. It is most commonly
used with hypervisors. It is one of the critical pieces in an SDN solution. If 
you are hearing about OpenVswitch for the first time then watch this [video](https://www.youtube.com/watch?v=rYW7kQRyUvA)
to learn more about it.

### Overlay Network

If you have a data center wherein your nodes are in different subnets then how do
you connect your vms in those nodes? The answer to this question is overlay networks.
An overlay network is built using tunneling. The tunnel could of type GRE or VXLAN.
When a vm in one node wants to send a packet to vm in another node, the openvswitch
encapsulates the packet with ip address of the node as an udp packet. Here is an
image of packet which is sent. Learn more about overlay [here](https://www.youtube.com/watch?v=Jqm_4TMmQz8)

<figure>
<a href="/img/vxlan.png"><img src="/img/vxlan.png"></a>
<figcaption>Packet flow</figcaption>
</figure>

### Linux network namespaces

You must have heard about namespaces in programming languages. They are used to
isolate variable names in different packages/libraries. So now think what could
be the variables in a network namespace in context of a operating system. You
guessed it right: `interfaces`, `routes`, `iptables`. The important thing to note
here is you can set a namespace as the primary namespace for a process in Linux.
To learn more about network namespaces, head over to [here](http://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/).

### Linux bridge

They are an age old concept. You can connect two interfaces using a linux bridge basically.
To learn more about it go [here](http://google.com).

### Veth pair

A veth pair is a tube basically. What goes in from one side comes out of the other side.

## What happens when you create a network

A namespace corresponding to this network is created in the network node or wherever
the neutron service is running. A dnsmasq process is started and attached to this
namespace. Whenver a vm uses an address from this network to boot up, the dnsmasq
process acts as a DHCP server and hands out an ip address to the vm.

## What happens when you boot a machine

A tap interface for the vm is created which gets an ip address from the dnsmasq
process. A linux bridge and a veth pair is also created. One end of the veth pair
and tap interface get connected with linux bridge. The other end of veth pair is
attached to an ovs-bridge. The bridge is commonly named as br-int(integration bridge).

## What happens when you create an external network
