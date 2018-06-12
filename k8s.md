# Kubernetes networking

- [Intra-pod networking](#intra-pod-networking)
- [Inter-pod networking](#inter-pod-networking)
- [Ingress and egress](#ingress-and-egress)

## Intra-pod networking

For this first exercise you'll be using [this Katacoda playground](https://katacoda.com/courses/kubernetes/playground).

Create a pod with two containers (a long-running app and a shell):

```bash
$ kubectl apply -f http://mhausenblas.info/container-networking/manifests/pod.yaml
$ watch kubectl get po
```

When you see the pod status `Running` (ca. after a minute), you can use the `shell` container in the pod to query the `app` container (serving on port `9876`):

```bash
$ kubectl exec somepod -it -c=shell bash
[root@somepod /]# curl localhost:9876/endpoint0
```

## Inter-pod networking

Using [this Katacoda scenario](https://katacoda.com/courses/kubernetes/networking-introduction) you will create Kubernetes services of type "ClusterIP", "Target Ports", "NodePort", "External IPs" as well as "Load Balancer" and explore them.

## Ingress and egress

Using [this Katacoda scenario](https://katacoda.com/courses/kubernetes/create-kubernetes-ingress-routes) you will deploy an Ingress controller and an Ingress resource that you'll be using to route traffic from the outside world to different services.