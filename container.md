# Container networking basics

## Namespaces and cgroups

Using [this Katacoda playground](https://www.katacoda.com/mhausenblas/scenarios/container-networking).

```bash
$ ip netns add east
$ ip netns add west
$ ip netns ls
$ ip netns exec east ip addr
$ ip netns exec east ip link set lo up
$ ip netns exec east ip addr
```

And explore via [cinf](https://github.com/mhausenblas/cinf).

## Single host

Using [this Katacoda playground](https://katacoda.com/courses/docker/playground).


```bash
$ docker network list
```

### Bridge mode

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

```bash
$ docker run -d --name=webserver --net=host nginx:1.9
$ curl docker:80
```

### Container mode

```bash
$ docker run -d --name=webserver nginx:1.9 
$ docker inspect webserver | grep IPAddress
$ docker run -it --name=shell --net=container:webserver centos:7 
$ [root@354e1e47323c /]# yum install iproute -y
$ [root@354e1e47323c /]# ip addr show
```

### No networking

```bash
$ docker run -d -P --name=webserver --net=none nginx:1.9
$ docker inspect webserver | grep IPAddress
```

### Custom network

https://katacoda.com/courses/docker/networking-intro


## Multi-host

https://katacoda.com/courses/weave/hello-net