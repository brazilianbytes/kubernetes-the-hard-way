# Provisioning Compute Resources

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are ultimately run. 

In this lab, you will set up your Raspberry PI 4 devices, put them on the network and make them ready to start.

In my lab, I expose my Kubernetes Controller Device to the internet using a Virtual Server from my TP-Link Home Router. Using my DDNS configuration, I use the domain name EXTERNAL_IP, like the GCP Load Balancer works.

This lab is simpler than the original and should be used to have fun!

![solutiondesign](images/terminator-screenshot.png)


## Networking

The Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model) assumes a flat network in which containers and nodes can communicate with each other. In cases where this is not desired [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) can limit how groups of containers are allowed to communicate with each other and external network endpoints.

> Setting up network policies is out of scope for this tutorial.

### Virtual Private Cloud Network

In this section a dedicated [Virtual Private Cloud](https://cloud.google.com/compute/docs/networks-and-firewalls#networks) (VPC) network will be setup to host the Kubernetes cluster.

Create the `kubernetes-the-hard-way` custom VPC network:

```
gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
```

A [subnet](https://cloud.google.com/compute/docs/vpc/#vpc_networks_and_subnets) must be provisioned with an IP address range large enough to assign a private IP address to each node in the Kubernetes cluster.

Create the `kubernetes` subnet in the `kubernetes-the-hard-way` VPC network:

```
gcloud compute networks subnets create kubernetes \
  --network kubernetes-the-hard-way \
  --range 10.240.0.0/24
```

> The `10.240.0.0/24` IP address range can host up to 254 compute instances.

### ~~Firewall Rules~~


### ~~Kubernetes Public IP Address~~ DDNS and Virtual Server

~~Allocate a static IP address that will be attached to the external load balancer fronting the Kubernetes API Servers~~

I configured my TP-Router to expose the Controller Node to internet using DDNS and Virtual Servers. The config is very different depending of your router brand or network configuration.

## ~~Compute Instances~~ Raspberry PI

The compute instances in this lab will be provisioned using [Raspberry PI OS Lite](https://www.raspberrypi.com/software/operating-systems/) but the unlisted 64 Bit. Each instance will be set up with a fixed private IP address to simplify the Kubernetes bootstrapping process.

### Raspberry OS 64 Bit

I used the brand new Raspberry OS Lite based on Debian Bullseye, but the 64-bit arm version. You can download it from [here](https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2021-11-08/2021-10-30-raspios-bullseye-arm64-lite.zip).

After this, you can write the image to SD Card using the Raspberry PI Imager using this [tutorial](https://www.raspberrypi.com/news/raspberry-pi-imager-imaging-utility/). When you choose the image, choose 'Choose Custom' and point to the zip image file that you downloaded before.

![Raspberry PI Imager screenshot](images/raspberry-pi-imager.png)

### Common Config

There are few customizations to do at the first boot for each RPi. 

> The default user is `pi` and the password is `raspberry`.

After boot your RPi, you need to use the raspi-config command:

```
sudo raspi-config
```

![raspi-config main menu](images/raspi-config-main.png)

You should change these options:
* 3 Interface Options > I2 SSH > Enable
* 4 Performance Options > P2 GPU Memory > 16 (More RAM for us!)
* 6 Advanced Options > A1 Expand FileSystem (More space to your SD card)

To set the machine name, use this command:

```
hostnamectl set-hostname <name>
```

You need to use static IPs to your RPIs. You can config static IP using Debian or define it using your Router and Mac Addresses.

> Config static IP in Debian not work for me for unknown reason. I fix then it using my router. This is better if you use the Virtual Server like me. :-)

At the end, we need to config these fixed IPs to /etc/hosts file (each node and your computer):

```
192.168.0.250   k8s-master
192.168.0.251   k8s-node-001
192.168.0.252   k8s-node-002
192.168.0.253   k8s-node-003

```

One last config is cgroups. To enable Raspberry PI cgroups, you need to put these config at the final of first line of `/boot/cmdline.txt`

```
... cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1
```

Check cgroups is working with this command:

```
cat /proc/cgroups
```

output (it's ugly, don't blame me)
```
#subsys_name	hierarchy	num_cgroups	enabled
cpuset	0	  242	1
cpu	    0	  242	1
cpuacct	0	  242	1
blkio	0	  242	1
memory	0	  242	1
devices	0	  242	1
freezer	0	  242	1
net_cls	0	  242	1
perf_event	0	242	1
net_prio	0	242	1
pids	0	242	1

```
> the memory line needs to has a '1'

### Disable Swap

To permanently disable the swap on Debian Bullseye, run the following commands:

```
sudo dphys-swapfile swapoff
sudo systemctl disable dphys-swapfile.service
```

## Kubernetes Controllers

~~Create three compute instances which will host the Kubernetes control plane:~~

Configure one of RPI devices to be the Controller. I named it as k8s-master.


## Kubernetes Workers

Configure the other three of RPI devices to be the Worker nodes. I named it as k8s-node-001, k8s-node-002 and k8s-node-003.

## Configuring SSH Access

SSH will be used to configure the controller and worker instances. 
Test SSH access to the `k8s-master` compute instances:

```
ssh pi@k8s-master
```

After the SSH ip fingerprint have been accepted you'll be logged into the `k8s-master` instance:

```
Linux k8s-master 5.10.63-v8+ #1459 SMP PREEMPT Wed Oct 6 16:42:49 BST 2021 aarch64
...
```

Use `Ctrl+D` or `exit` at the prompt to exit the `k8s-master` compute instance:

```
pi@k8s-master:~$ exit
```
Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
