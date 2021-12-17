---
author: Jason King <jason.brian.king@gmail.com>
state: predraft
---

# IPD vsock AF_VSOCK Support for illumos

## Introduction

VM Sockets were a socket facility introduced by VMware circa 2013.
The concept was to provide a bi-directional communication channel between the
hypervisor and a VM guest in the form of a new socket family (`AF_VSOCK`) and
new socket address (`struct sockaddr`) type (`struct sockaddr_vm`).
Various services could then be built on top of this communication channel.
Examples of this include facilites for auto configuration of guests (via the
exchange of metadata between the hypervisor and guest), file sharing via hgfs,
etc.

Eventually, this concept was adopted by the virtio standard as the virtio
socket device[virtio](https://docs.oasis-open.org/virtio/virtio/v1.1/csprd01/virtio-v1.1-csprd01.html#x1-39000010).
This IPD proposes implementing `AF_VSOCK` on illumos.

## API


# illumos as a guest

When illumos is running as a guest, this proposal is fairly straightforward.
illumos already provides a (non-public) interface for 
