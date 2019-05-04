+++
Tags = ["DevOps","Docker","Podman"]
Description = "How to move from Docker to Podman"
date = "2019-04-05T14:02:30+01:00"
title = "From Docker To Podman"
Categories = ["Container","DevOps"]

+++

# HowTo: Switch from Docker to Podman

TL;DR; I changed, feel more save and have less problems.


Recently I heared about [Podman](https://podman.io/), an alternative to [Docker](https://www.docker.com/) which is more like [Kubernetes](https://kubernetes.io/) than Docker is. This you can see already in the name **pod**man which shows that it's working with pods as Kubernetes does.

Why is this interresting?
Because out in the wild normally a fully configurable and stackable system is used for containers. But this is not about Kubernetes.

Second interresting part of [Podman](https://podman.io/) is, that it runs in Userspace and not as a deamon as root.
This is a big security advantage and also gives me as a user more possibilities: Linked storages are under my control and not owned by root. So I not have to start a container, link in the storage and run the delete command - I just can delete the content from my host.

As third: It's fully compatible with [OCI - Open Container Initiative](https://www.opencontainers.org/), so you can run every Docker-Image which keeps up with those standards.

What's a drawbak: No support yet (May 2019) for Windoes and OSX.

## Installation

Most distributions have a Podman package. Due to the fact I recently switched to [archlinux](https://www.archlinux.org/) I will show you how it's done there.

Remove docker and install podman:

```bash
$ sudo pacman -R docker docker-compose python-dockerpty python-docker python-docker-pycreds
$ sudo pacman -S podman podman-docker
```
There you go, all is installed and ready for use.


## Configuration

When you firts try to use it, you will notice some warnings like this:

```bash
$ podman container list
WARN[0000] cannot find mappings for user lukas: No subuid ranges found for user "lukas" in /etc/subuid 
WARN[0000] using rootless single mapping into the namespace. This might break some images. Check /etc/subuid and /etc/subgid for adding subids 
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES

```
This error comes from [User Namespaces](https://lwn.net/Articles/532593/) in the Linux Kernel: User Namespaces allows a process to see a completely different uid/gid mapping than the system.

The to files containes as first the suername/groupname, the starting uid/gid and as last the number of available uid/gid in the process.

I have something like this on my system: This gives my pods the uids/gids 1100000 up to 1165536.
```bash
$ cat /etc/subuid 
lukas:110000:65536

$ cat /etc/subgid 
lukas:110000:65536
```

Lets see if this works:
```bash
$ cat /proc/self/uid_map 
         0          0 4294967295

$ podman run fedora-minimal cat /proc/self/uid_map
...
         0       1000          1
         1     110000      65536
```

* The First line shows the uid-Mapping on the base system:  UID 0 is UID 0 and the system has a max of 4294967295 uids.
* The Second line shows the mapping in the container:  UID 0 is UID 1000 on the host; The container as has only UIDs from 1 up to 65536.

See more about the user namespace in podman on this [blog post](https://www.projectatomic.io/blog/2018/05/podman-userns/).

For Archlinux also read about [Enable support to run unprivileged containers](https://wiki.archlinux.org/index.php/Linux_Containers).


## Conclusion

Easy installation, no change for a docker user on the CLI, more security and it feels faster.

What can be triky is the change from Docker to Podman with existing data this are owned by root and newly have to be owned by you. One HBase based system (openTSDB) I started over which was more easy than change all settings and permissions.
