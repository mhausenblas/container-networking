# Container networking basics

Using [Katacoda](https://katacoda.com/courses/docker/playground).

```bash
$ docker network list
```

## Bridge mode

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

## Host mode

```bash
$ docker run -d --name=webserver --net=host nginx:1.9
$ curl docker:80
```

## Container mode

```bash
$ docker run -d --name=webserver nginx:1.9 
$ docker inspect webserver | grep IPAddress
$ docker run -it --name=shell --net=container:webserver centos:7 
$ [root@354e1e47323c /]# yum install net-tools -y
$ [root@354e1e47323c /]# ifconfig
```

## No networking

```bash
$ docker run -d -P --name=webserver --net=none nginx:1.9
$ docker inspect webserver | grep IPAddress
```