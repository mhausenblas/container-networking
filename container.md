# Container networking basics

- [Linux namespaces and cgroups](#linux-namespaces-and-cgroups)
- [Single host](#single-host)
- [Multi-host](#multi-host)

## Linux namespaces and cgroups

Using [this Katacoda playground](https://www.katacoda.com/mhausenblas/scenarios/container-networking) you will create two network namespaces and set up a communication policy.

First, create the two Linux network namespaces `east` and `west` and explore them:

```bash
$ ip netns add east
$ ip netns add west
$ ip netns ls
$ ip netns exec east ip addr
$ ip netns exec east ip link set lo up
$ ip netns exec east ip addr
$ ip netns exec west ip route show
```

Now, configure the namespace `east`:

```bash
$ ip link add east0 type veth peer name east1
$ ip link set east1 netns east up
$ ip addr add 10.200.1.1/24 dev east0
$ ip link set east0 up
$ ip netns exec east ip addr add 10.200.1.2/24 dev east1
$ ip netns exec east ip link set east1 up
$ ip netns exec east ip link set lo up
$ ip netns exec east ip route add default via 10.200.1.1
```

Now enable network access from the `east` namespace to the host (root namespace):

```bash
$ echo 1 > /proc/sys/net/ipv4/ip_forward
$ iptables -P FORWARD DROP && iptables -F FORWARD && iptables -t nat -F
$ iptables -t nat -A POSTROUTING -s 10.200.1.0/255.255.255.0 -o ens3 -j MASQUERADE
$ iptables -A FORWARD -i ens3 -o east0 -j ACCEPT
$ iptables -A FORWARD -o ens3 -i east0 -j ACCEPT
$ ip netns exec east ip route sh
```

Next, let's launch a very simple webserver in the `east` namespace:

```bash
$ echo "I am serving from the East" > index.html
$ echo "while true ; do nc -l 80 < index.html ; done" > webserver.sh
$ chmod 750 webserver.sh
$ ip netns exec east ./webserver.sh
```

Now we can query it from the `root` namespace:

```bash
$ ip netns exec east ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
4: east1@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether ea:dd:40:79:95:bf brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.200.1.2/24 scope global east1
       valid_lft forever preferred_lft forever
    inet6 fe80::e8dd:40ff:fe79:95bf/64 scope link
       valid_lft forever preferred_lft forever
$ curl 10.200.1.2
I am serving from the East
```

You can now access the webserver from the host, but not from the `west` namespace. Let's verify this:

```bash
$ ip netns exec west curl 10.200.1.2
```

Bonus:

- Check network stats with `ip netns exec east netstat -i`.
- Explore namespace using [cinf](https://github.com/mhausenblas/cinf).

## Single host

For all below (single host) exercises you'll be using [this Katacoda playground](https://katacoda.com/courses/docker/playground).

Check what is there by default:

```bash
$ docker network list
```

### Bridge mode

Create a container running a webserver using bridge mode and query it from another container:

```bash
$ docker run -d -P --name=webserver --net=bridge nginx:1.9
$ docker inspect webserver | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.18.0.2",
                    "IPAddress": "172.18.0.2",
$ docker run -it --name=shell centos:7
$ [root@f106f4da3df5 /]# curl 172.18.0.2
$ docker kill webserver && docker rm webserver && docker rm shell
```

### Host mode

Create a container running a webserver using host mode and query it:

```bash
$ docker run -d --name=webserver --net=host nginx:1.9
$ curl docker:80
$ docker kill webserver && docker rm webserver
```

### Container mode

Create a container running a webserver and re-use the network in another container:

```bash
$ docker run -d --name=webserver nginx:1.9 
$ docker inspect webserver | grep IPAddress
$ docker run -it --name=shell --net=container:webserver centos:7 
$ [root@354e1e47323c /]# yum install iproute -y
$ [root@354e1e47323c /]# ip addr show
$ docker kill webserver && docker rm webserver && docker rm shell
```

### No networking

Verify that if no networking is specified, we indeed have no connectivity:

```bash
$ docker run -d -P --name=webserver --net=none nginx:1.9
$ docker inspect webserver | grep IPAddress
```

### Custom network

Using [this Katacoda scenario](https://katacoda.com/courses/docker/networking-intro) you will create a custom single-host container network and perform several actions in it.


## Multi-host

Using [this Katacoda scenario](https://katacoda.com/courses/weave/hello-net) you will create a multi-host container network with [Weave Net](https://github.com/weaveworks/weave/).