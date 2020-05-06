# Kubernetes Basics

This repo constains some of the most used commands on Kubernetes and the basics to  get started with it.

## Kubectl autocomplete

This is realy usefull when using the command kubectl. It enables the autocomplete for the command.

``` bash
source <(kubectl completion bash) # setup autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
```

## Most used commands to inspect your resources and cluster

This command describe cluster information and current evets. It shows erros like memory issues and others occuring inside nodes.
```bash
kubectl describe nodes
```

This command describes information about the pod named memory-demo-3 inside the namesace mem-example. Its usefull to check the current state of the Pod. It lists all events of the Pod.
``` bash
kubectl describe pod memory-demo-3 --namespace=mem-example
```
## Cluster management

### Manage resources requests

Memory requests and limits are associated with Containers, but it is useful to think of a Pod as having a memory request and limit. The memory request for the Pod is the sum of the memory requests for all the Containers in the Pod. Likewise, the memory limit for the Pod is the sum of the limits of all the Containers in the Pod.

CPU requests and limits are associated with Containers, but it is useful to think of a Pod as having a CPU request and limit. The CPU request for a Pod is the sum of the CPU requests for all the Containers in the Pod. Likewise, the CPU limit for a Pod is the sum of the CPU limits for all the Containers in the Pod.

### Quality of Service - QoS

Kubernetes automatically assigns a QoS to a Pod when it runs. Available types of QoS are:

- Guaranteed
- BestEffort
- Burstable

The selection is based on requested resources for a Pod.

#### QoS Guaranteed

For a Pod to be given a QoS class of Guaranteed:

- Every Container in the Pod must have a memory limit and a memory request, and they must be the same.
- Every Container in the Pod must have a CPU limit and a CPU request, and they must be the same.

#### QoS Burstable

A Pod is given a QoS class of Burstable if:

- The Pod does not meet the criteria for QoS class Guaranteed.
- At least one Container in the Pod has a memory or CPU request.

#### QoS BestEffort

For a Pod to be given a QoS class of BestEffort, the Containers in the Pod must not have any memory or CPU limits or requests.

## Storage

### Persistent Volume

A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster just like a node is a cluster resource. PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV. This API object captures the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system.

#### Persistent Volume Classes

A PV can have a class, which is specified by setting the storageClassName attribute to the name of a StorageClass. A PV of a particular class can only be bound to PVCs requesting that class. A PV with no storageClassName has no class and can only be bound to PVCs that request no particular class.

A class could be created by the user, and binds directelly to one of the PV types.
When creating a new PV one could specify the desired storage class, automatically configured by the StorageClass object.


#### Persistent Volume Types

- GCEPersistentDisk
- AWSElasticBlockStore
- AzureFile
- AzureDisk
- CSI
- FC (Fibre Channel)
- FlexVolume
- Flocker
- NFS
- iSCSI
- RBD (Ceph Block Device)
- CephFS
- Cinder (OpenStack block storage)
- Glusterfs
- VsphereVolume
- Quobyte Volumes
- HostPath (Single node testing only – local storage is not supported in any way and WILL NOT WORK in a multi-node cluster)
- Portworx Volumes
- ScaleIO Volumes
- StorageOS

#### Persistent Volume Modes

- ReadWriteOnce – the volume can be mounted as read-write by a single node
- ReadOnlyMany – the volume can be mounted read-only by many nodes
- ReadWriteMany – the volume can be mounted as read-write by many nodes

### Persistent Volume Claim

A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., they can be mounted once read/write or many times read-only).

To enable dynamic storage provisioning based on storage class, the cluster administrator needs to enable the DefaultStorageClass admission controller on the API server. This can be done, for example, by ensuring that DefaultStorageClass is among the comma-delimited, ordered list of values for the --enable-admission-plugins flag of the API server component. For more information on API server command-line flags, check kube-apiserver documentation.

### Secrets

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in an image. Users can create secrets and the system also creates some secrets.

To use a secret, a Pod needs to reference the secret. A secret can be used with a Pod in two ways:

- As files in a volume mounted on one or more of its containers.
- By the kubelet when pulling images for the Pod.

The Secret contains two maps: **data** and **stringData**. The data field is used to store arbitrary data, encoded using base64. The stringData field is provided for convenience, and allows you to provide secret data as unencoded strings.

You can set the file access permission bits for a single Secret key. If you don’t specify any permissions, 0644 is used by default. You can also set a default mode for the entire Secret volume and override per key if needed.

FEATURE STATE: Kubernetes v1.18 

The Kubernetes alpha feature Immutable Secrets and ConfigMaps provides an option to set individual Secrets and ConfigMaps as immutable. For clusters that extensively use Secrets (at least tens of thousands of unique Secret to Pod mounts), preventing changes to their data has the following advantages:

protects you from accidental (or unwanted) updates that could cause applications outages
improves performance of your cluster by significantly reducing load on kube-apiserver, by closing watches for secrets marked as immutable.

#### Secret as Env

When using a secret as an environment variable its not necessary to declare is as a volume. Just point to the created secret resource name.

### Config Map

A config map is a structure to store key value pairs in Kubernets. It could be consumed as Envs or files.

Config Maps are automatically updated.

There are several ways to create a ConfigMap over kubectl create command, for example:

Creates a ConfigMap based on configurations inside the properties file.
```bash
kubectl create configmap <config-map-name> --from-file=path/to/file.properties
```

Creates a ConfigMap for multiple files
```bash
kubectl create configmap <config-map-name> --from-file=path/to/file properties --from-file=path/to/other/file.properties
```

Creates a ConfigMap for all files in the tree.
```bash
kubectl create configmap <config-map-name> properties --from-file=path/to/files/
```

It's possible to use yaml to create ConfigMap objects but it does not support geting data directelly from files

## Services

A service makes interface to connect to a set of Pods. It's binded via selectors.
A Service Could map multiple ports and protocols at the same time.

A service is composed by two main blocks, the Service properly and an Endpoint.

The service describes wich port and portocol will be exposed and the type of the LoadBalancer, but the Endpoint defines with is the target to bind, it means wich is the IP and port from from origin to map. This allows Kubernetes to use service to Map external services on VMs for example.
To use a service with a custom Endpoint dont apply any selector on service creation, so it wont create an Endpoint automatically.

Service for internal pods:
- Uses selectors to define wich Pods to map in Service creation. It will automatically create a new Endpoint

Service for external services
- Create a Service without setting selectors and then create an Endpoint object with the target specification.

A best alternative for scalability is using **EndpointSlices**.

For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, that’s outside of your cluster.

Kubernetes ServiceTypes allow you to specify what kind of Service you want. The default is ClusterIP.

Type values and their behaviors are:

*ClusterIP*: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.
*NodePort*: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>.
*LoadBalancer*: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
*ExternalName*: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record

with its value. No proxying of any kind is set up.

## Ingress

An API object that manages external access to the services in a cluster, typically HTTP.

Ingress may provide load balancing, SSL termination and name-based virtual hosting.

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

An ingress acts as a layer to connect external internet inside the cluster.

You must have an ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect.

You may need to deploy an Ingress controller such as ingress-nginx. You can choose from a number of Ingress controllers.

Ideally, all Ingress controllers should fit the reference specification. In reality, the various Ingress controllers operate slightly differently.

[Ingress controller types](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

IMPORTANT:

To use an ingress you MUST have a LoadBalancer provider of any kind. Ingresses uses LoadBalancer to create a public access IP and exposes the application to public network.

On Minikube you MUST enable the ingress addon, that installs the nginx ingress controller.