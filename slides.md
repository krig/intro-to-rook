# Introducing
# Rook

<img src="img/index-hero.svg" class="plain" style="max-height:400px;">

<div style="font-size:0.5em;">Kristoffer Gr&ouml;nlund &lt;kgronlund@suse.com&gt;</div>

Note:
About me; Work for SUSE; Storage team on rook

---

Ceph &bull; <b>Rook</b> &bull; Kubernetes

Note:

Going to talk about three different things: Ceph, Rook and Kubernetes.
Anyone unfamiliar with all of these? Won't be able to dig deep.

---

# Software Defined Storage

Note:

Ceph is a Software Defined Storage solution.
Contrasted against buying a storage array from a vendor.
Pros: Cheaper, more flexible, adaptible, cloud compatible.
Cons: More complexity to handle.

---

<img src="img/ceph.png" class="plain" style="max-height:550px;float:left;">

&nbsp;

&nbsp;

* Open source
* Distributed
* Massively scalable
* Self healing
* Runs on commodity hardware / public cloud

Note:

Ceph
* Open source
* Distributed
* Massively scalable
* Self healing
* Runs on commodity hardware / public cloud


---

<img src="img/ses.png" class="plain" style="max-height:550px;">

Note:

# Flexible API options
* Object (`radosgw`)
  * REST
  * S3
  * SWIFT
* Block (`rbd`)
* File (`cephfs`)

---

### No Single Point Of Failure

<img src="img/ceph-architecture.png" class="plain" style="max-height:500px;">

Note:

Architecture:
  * CRUSH algorithm - automatic balancing
  * OSD: Manages a single device
  * MON: Communication coordinators
  * MDS: Metadata for the file systems
  * RGW: RADOS Gateway, REST / S3 / ...
  * MGR: Manager, dashboards

---

<img src="img/dashboard.png" class="plain" style="max-height:700px">

Note:

Comes with user-friendly tools,
management dashboard, prometheus
integration, etc.

---

# Kubernetes

<img src="img/kubernetes.svg" class="plain" style="max-height:500px;">

Note:

Container orchestration platform.
Automate application deployment, scaling and management.

Handles service discovery, load balancing, rollouts and rollbacks.

Is self-healing.

Handles secrets and configuration management.

---

# Concepts

* Pod
* Service
* Namespace
* Controller
* Volume
* Custom Resource Definition (CRD)

Note:

Pod = basic execution unit of an app. One or more containers with resources like
storage, networking, runtime options.

Service = interface to an app, DNS name for a set of pods plus load balancing.

Namespace = organize the cluster.

Controllers create and manage replicated sets of Pods (DaemonSet, ReplicaSet).

Volume = Directory accessible by a Pod.

CRD = An interface for extending Kubernetes with new types.

---

# Kubernetes Storage Story

1. Volume Plugins

2. FlexVolume *

3. CSI *

Note:

how does storage work in Kubernetes? How can services access and store data?
Not going to talk about using databases here.
Kubernetes designed for stateless services.
Volume plugins provide interface to storage somewhere else.
Kubernetes assumes someone else sorts out storage.

---

<img src="img/disclaimer.gif" class="plain" style="max-height:400px;float:right;">

# Challenges
#### External Storage / Cloud Vendor

* Vendor lock-in
* Portability
* Connectivity
* Deployment
* Administration

Note:
You might say just use EBS or whatever storage the cloud vendor provides.
There are some issues with that though.
A big one (surprisingly) is cost.
Having storage "somewhere else" means dealing with connectivity between storage
and the applications. Deployment burden. Who manages this thing?

---

<img src="img/index-what-is-rook.svg" class="plain" style="height:600px;">

Note:

This is where Rook comes in.
Storage Orchestration Framework.
Extends Kubernetes with new primitives.
Configures and manages storage provides in Kubernetes,
and exposes storage to pods.

---

<img src="img/provision.svg" class="plain" style="float:right;height:450px;">

### Multiple storage providers

* **Ceph**
* **EdgeFS**
* CockroachDB
* Cassandra
* NFS
* Yugabyte DB
* Apache Ozone

Note:
Rook is a framework for multiple storage systems.
Ceph and EdgeFS are stable, the rest are either
in beta or alpha state as of rook 1.2. 

---

# Rook Operator

* Bootstrap and Monitor
* Manages Ceph
* Creates Agents

Note:
* Bootstraps and monitors the storage cluster
* Manages Ceph MONs and a DaemonSet for OSDs
* CRDs for pools, object stores and file systems
* Creates Rook agents

---

# Rook Agents

* Runs on all nodes
* Handles storage operations

Note:
* Deployed on every Kubernetes node
* Handles storage operations
  * Attach network storage
  * Mount volumes
  * Format file systems

---

* Persistent Volume (PV)
* Persistent Volume Claim (PVC)
* Storage Classes

Note:

* **Persistent Volume (PV)**
  * Actual slice of storage
  * Manually provisioned by administrator, or
  * Dynamically provisioned by Storage Classes
  * Independent lifetime from Pods

* **Persistent Volume Claim (PVC)**
  * Request for storage
  * Consumes PV resources
  * Can request specific size, access mode
  
* **Storage Classes**
  * Defines types of storage available
  * Enable dynamic storage provisioning
  * Cluster creates and assigns PVs to PVCs

---

<img src="img/rook-architecture.png" class="plain" style="height:600px;">

Note:

Architecture overview.
Important parts have blue background:
Rook manages the storage provider (ceph)
Exposes storage to pods.

---

# Use Case: Pure Ceph Cluster

* Dedicated K8S cluster for running Ceph
* Can serve multiple K8S application clusters
* Not common

---

# Use Case: Shared Cluster

* One partitioned K8S cluster
* Dedicated storage nodes for Ceph
* Dedicated compute nodes for Apps

---

# Use Case: Unified Cluster

* All nodes run both Ceph and workloads
* Most common

---

# Use Case: External Cluster

* Rook as interface to external Ceph cluster
* Added in Rook v1.1

---

### Try it out

<asciinema-player id="terminal-player" src="/js/ceph.cast" cols="61" rows="18"
theme="monokai" font-size="big"></asciinema-player>

---

<pre class="stretch">
<code data-trim class="hljs yaml">
# cat object.yaml
apiVersion: ceph.rook.io/v1
kind: CephObjectStore
metadata:
  name: my-store
  namespace: rook-ceph
spec:
  metadataPool:
    failureDomain: host
    replicated:
      size: 3
  dataPool:
    failureDomain: host
    erasureCoded:
      dataChunks: 2
      codingChunks: 1
  preservePoolsOnDelete: true
  gateway:
    type: s3
    sslCertificateRef:
    port: 80
    securePort:
    instances: 1
</code>
</pre>

Note:
Create an object store

---

```sh
# Create the object store
kubectl create -f object.yaml

# To confirm the object store is configured, wait for the rgw pod to start
kubectl -n rook-ceph get pod -l app=rook-ceph-rgw
```

---

rook.io

Note:

There's a lot more to it, for more info and docs see rook.io

---

# Conclusion

<img src="img/opensource.svg" class="plain" style="height:450px;">

Note:

why use Rook? If you need Storage and are running Kubernetes,
there really isn't anything else that compares.

Provides Storage within Kubernetes. Consistent interface wherever Kubernetes
runs, different cloud providers, on premise, etc.

Version 1.2.1 released recently. Ceph backend most mature.
Ceph itself is proven technology, running in production since 2012.

Avoid lock-in, gain scalability, flexibility.

---

<img src="img/suse.svg" class="plain" style="height:300px;" >

**suse.com/storage**

<hr>

* SUSE Storage 6
  * Technology Preview (feedback welcome)
* SUSE Storage 7
  * Rook on CaaSP as supported stack

Note:

For a supported option, SUSE has a tech preview out now, and in
the next release SES will be fully supported on CaaSP, the SUSE K8S
distribution.
