# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

> There are [other ways](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) to implement the Kubernetes networking model.

## The Routing Table

~~In this section you will gather the information required to create routes in the `kubernetes-the-hard-way` VPC network.~~

Execute the commands to create route table for each instance (controller and workers):

```
  sudo ip route add 10.200.3.0/24 via 192.168.0.253
  sudo ip route add 10.200.2.0/24 via 192.168.0.252
  sudo ip route add 10.200.1.0/24 via 192.168.0.251
```

> output

```
10.240.0.20 10.200.0.0/24
10.240.0.21 10.200.1.0/24
10.240.0.22 10.200.2.0/24
```


Next: [Deploying the DNS Cluster Add-on](12-dns-addon.md)
