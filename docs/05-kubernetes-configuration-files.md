# Generating Kubernetes Configuration Files for Authentication

In this lab you will generate [Kubernetes configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), also known as kubeconfigs, which enable Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

## Client Authentication Configs

In this section you will generate kubeconfig files for the `controller manager`, `kubelet`, `kube-proxy`, and `scheduler` clients and the `admin` user.

### Kubernetes Public IP Address

Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

Retrieve the `kubernetes-the-hard-way` static IP address:

```
KUBERNETES_PUBLIC_ADDRESS="$(gcloud compute addresses describe kubernetes-the-hard-way \
  --format 'value(address)')"
```

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/reference/access-authn-authz/node/).

> The following commands must be run in the same directory used to generate the SSL certificates during the [Generating TLS Certificates](./04-certificate-authority.md) lab.

Generate a kubeconfig file for each worker node:

```
for instance in worker-0 worker-1 worker-2; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority ca.pem \
    --embed-certs \
    --kubeconfig "${instance}.kubeconfig" \
    --server "https://${KUBERNETES_PUBLIC_ADDRESS}:6443"

  kubectl config set-credentials "system:node:${instance}" \
    --client-certificate "${instance}.pem" \
    --client-key "${instance}-key.pem" \
    --embed-certs \
    --kubeconfig "${instance}.kubeconfig"

  kubectl config set-context default \
    --cluster "kubernetes-the-hard-way" \
    --kubeconfig "${instance}.kubeconfig" \
    --user "system:node:${instance}"

  kubectl config use-context default \
    --kubeconfig "${instance}.kubeconfig"
done
```

Results:

```
worker-0.kubeconfig
worker-1.kubeconfig
worker-2.kubeconfig
```

### The kube-proxy Kubernetes Configuration File

Generate a kubeconfig file for the `kube-proxy` service:

```
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority ca.pem \
  --embed-certs \
  --kubeconfig kube-proxy.kubeconfig \
  --server "https://${KUBERNETES_PUBLIC_ADDRESS}:6443"

kubectl config set-credentials system:kube-proxy \
  --client-certificate kube-proxy.pem \
  --client-key kube-proxy-key.pem \
  --embed-certs \
  --kubeconfig kube-proxy.kubeconfig

kubectl config set-context default \
  --cluster kubernetes-the-hard-way \
  --kubeconfig kube-proxy.kubeconfig \
  --user system:kube-proxy

kubectl config use-context default \
  --kubeconfig kube-proxy.kubeconfig
```

Results:

```
kube-proxy.kubeconfig
```

### The kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `kube-controller-manager` service:

```
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority ca.pem \
  --embed-certs \
  --kubeconfig kube-controller-manager.kubeconfig \
  --server https://127.0.0.1:6443

kubectl config set-credentials system:kube-controller-manager \
  --client-certificate kube-controller-manager.pem \
  --client-key kube-controller-manager-key.pem \
  --embed-certs \
  --kubeconfig kube-controller-manager.kubeconfig

kubectl config set-context default \
  --cluster kubernetes-the-hard-way \
  --kubeconfig kube-controller-manager.kubeconfig \
  --user system:kube-controller-manager

kubectl config use-context default \
  --kubeconfig kube-controller-manager.kubeconfig
```

Results:

```
kube-controller-manager.kubeconfig
```


### The kube-scheduler Kubernetes Configuration File

Generate a kubeconfig file for the `kube-scheduler` service:

```
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority ca.pem \
  --embed-certs \
  --kubeconfig kube-scheduler.kubeconfig \
  --server https://127.0.0.1:6443

kubectl config set-credentials system:kube-scheduler \
  --client-certificate kube-scheduler.pem \
  --client-key kube-scheduler-key.pem \
  --embed-certs \
  --kubeconfig kube-scheduler.kubeconfig

kubectl config set-context default \
  --cluster kubernetes-the-hard-way \
  --kubeconfig kube-scheduler.kubeconfig \
  --user system:kube-scheduler

kubectl config use-context default \
  --kubeconfig kube-scheduler.kubeconfig
```

Results:

```
kube-scheduler.kubeconfig
```

### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority ca.pem \
  --embed-certs \
  --kubeconfig admin.kubeconfig \
  --server https://127.0.0.1:6443

kubectl config set-credentials admin \
  --client-certificate admin.pem \
  --client-key admin-key.pem \
  --embed-certs \
  --kubeconfig admin.kubeconfig

kubectl config set-context default \
  --cluster kubernetes-the-hard-way \
  --kubeconfig admin.kubeconfig \
  --user admin

kubectl config use-context default \
  --kubeconfig admin.kubeconfig
```

Results:

```
admin.kubeconfig
```

##

## Distribute the Kubernetes Configuration Files

Copy the appropriate `kubelet` and `kube-proxy` kubeconfig files to each worker instance:

```
for instance in worker-0 worker-1 worker-2; do
  gcloud compute scp \
    "${instance}.kubeconfig" \
    kube-proxy.kubeconfig \
    "${instance}:"
done
```

Copy the appropriate `kube-controller-manager` and `kube-scheduler` kubeconfig files to each controller instance:

```
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp \
    admin.kubeconfig \
    kube-controller-manager.kubeconfig \
    kube-scheduler.kubeconfig \
    "${instance}:"
done
```

Next: [Generating the Data Encryption Config and Key](./06-data-encryption-keys.md)
