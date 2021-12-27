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
The goal was to allow VM guest services to be built on top of this communication channel since it would provide a largely familiar programming API.
Examples of services using VM sockets include guest auto configuration (via the
exchange of metadata between the hypervisor and guest) and file sharing via hgfs.

This facility was adopted was adopted by the virtio standard as the virtio
socket device[virtio](https://docs.oasis-open.org/virtio/virtio/v1.1/csprd01/virtio-v1.1-csprd01.html#x1-39000010).
While conceptually similar to the original VMware `AF_VSOCK` guest<->hypervisor interface, the virtio vsock specification is noticably different.
For instance, the VMware interface supports both stream (`SOCK_STREAM`) and message (`SOCK_DGRAM`) based communication while the virtio socket currently only supports stream based communication.
Additionally, other hypervisors also support similar facilities.
For example, Hyper-V has a AF_HYPERV socket.
While AF_HYPERV uses GUIDs as addresses, there is a published method of mapping the AF_VSOCK cids to GUIDs as a compatability layer.

This IPD proposes implementing `AF_VSOCK` on illumos.
As multiple hypervisors provide a similar functionality (the virtio socket, VMware's vmci, etc), this implementation will be split into a generic front end that provides the socket endpoints (`struct XXX`) and a hypervisor specific backend as detailed below.

When the original VMCI interface was introduced by VMware, it was possible to use `AF_VSOCK` sockets for guests on the same host to communicate with each other.
Because of security concerns, this ability was later removed by VMware.
For clarity, this proposal is very explicitly only proposing communication between a host and a guest and _not_ between guests.

## Programming API

As noted above, this IPD proposes adding a new socket familiy `AF_VSOCK` as well as a new socket address structure `struct sockaddr_vm`.
The value of `AF_VSOCK` should be a currently unusued value, and we should strive to coordinate with any downstream distributions to make sure the value chosen does not present any conflicts.
AF_VSOCK sockets use 64-bit unsigned integers (called cids) as their address.
In most implementations, the upper 32-bits are reserved.
A number of 'well-known' cids are defined for most backends.

The definition of `struct sockaddr_vm` will be source compatible with that on Linux for ease of portability:

```
struct sockaddr_vm {
...
}
```

When illumos is running as a guest, usage is pretty straightforward.
The `socket(3SOCKET)` function is called to create an `AF_VSOCK` socket, and the usual `3LIBSOCKET` functions can be used to bind, connect, read, write to any host services as desired.

When illumos is operating as a host, the situation is more interesting.
Ideally, as on Linux, it would be ideal that host services only need to also create an `AF_VSOCK` socket, and act as the other end of the socket.


## Kernel Interfaces

Within the kernel, the support will be broken into several modules.
The first module (tentatively named `vsock`) provides the socket (XXX) entry points for an `AF_VSOCK` socket and acts as a socket provider (XXX).
This module will handle generic state management for a `AF_VSOCK` socket, as well as act as a backend when illumos is functioning as a hypervisor.

Each hypervisor will require its own backend module that handles the hypervisor specific communication.
The initial proposal will include backend support for a virtio socket device.
These hypervisor specific backend kernel modules will communicate with the `vsock` module using a private API between them.
When no backend module is available, any socket operations on a `AF_VSOCK` will return `EAFNOTSUPPORTED` unless a more appropriate error message exists.
In essence, it is returned whenever a request would otherwise succeed if a suitable backend existed.

XXX Insert proposed vsock<->backend API

### Nested virtualization


