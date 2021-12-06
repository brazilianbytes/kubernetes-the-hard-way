# Configuring kubectl for Remote Access

In this lab you will generate a kubeconfig file for the `kubectl` command line utility based on the `admin` user credentials.

> Run the commands in this lab from the same directory used to generate the admin client certificates.

## The Admin Kubernetes Configuration File

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

Generate a kubeconfig file suitable for authenticating as the `admin` user:

```
{
  KUBERNETES_PUBLIC_ADDRESS=k8s.solutionarchitect.rocks

  kubectl config set-cluster kubernetes-the-rpi-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem

  kubectl config set-context kubernetes-the-rpi-way \
    --cluster=kubernetes-the-rpi-way \
    --user=admin

  kubectl config use-context kubernetes-the-rpi-way
}
```

## Verification

Check the version of the remote Kubernetes cluster:

```
kubectl version
```

> output

```
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.4", GitCommit:"3cce4a82b44f032d0cd1a1790e6d2f5a55d20aae", GitTreeState:"clean", BuildDate:"2021-08-11T18:16:05Z", GoVersion:"go1.16.7", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479", GitTreeState:"clean", BuildDate:"2021-04-08T16:25:06Z", GoVersion:"go1.16.1", Compiler:"gc", Platform:"linux/arm64"}

```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

> output

```
NAME           STATUS   ROLES    AGE   VERSION
k8s-node-001   Ready    <none>   19m   v1.21.0
k8s-node-002   Ready    <none>   19m   v1.21.0
k8s-node-003   Ready    <none>   19m   v1.21.0
```

Next: [Provisioning Pod Network Routes](11-pod-network-routes.md)
