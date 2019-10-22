# Kubernetes

Kubernetes is an amazing tool that abstracts a group of computers into a single
cluster.  If you want to run a service, or multiple copies of that service, k8s
(k8s is an abbreviation for Kubernetes), will take care of figuring out which
computer to run it on.  If that computer goes down, k8s will automatically run
it on another.  It really feels like the "Borg" that one of its predecessors is
named after.

## Resources

- [RKE](https://github.com/rancher/rke) - A simple Kubernetes installer for 
"custom" configurations. Installing k8s can be difficult, and most tutorials
online assume that you're installing to cloud servers.  But I wanted to install
to VMs and linux servers I had in my apartment, which RKE did easily.
- [Kubernetes in Action
  (2017)](https://www.manning.com/books/kubernetes-in-action) - Book that I
  used to work through the initial concepts.  Recommended.

```
Created:       Tue 22 Oct 2019 06:46:45 AM CDT 
Last Modified: Tue 22 Oct 2019 06:46:45 AM CDT
```
