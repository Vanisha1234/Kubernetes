## Volumes In Kubernetes
### Docker Volumes
- Containers are short-lived. The data within a container is destroyed along with the container itself.
- Hence, to persist data, we use volumes.
- The data is stored in the volume so it remains available even if the container is deleted.
  
---

### In Kubernetes
- Similarly, Pods in Kubernetes are transient in nature, meaning data is lost when the Pod is deleted.
- To persist data even after a Pod is destroyed, we attach a volume to the Pod.
- Once a volume is created, it can be backed by a hostPath, which is a directory on the node.
- Any data written to the volume is stored in that directory on the node.
- To access the volume from within a container, we mount the volume to a directory inside the container.

> ⚠️ Storing volumes using hostPath is not ideal in a multi-node cluster.

---

### Volume Definition File Example
```bash
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
    - name: alpine
      image: alpine
      command: ["/bin/sh", "-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
      volumeMounts:                       # Mount the volume inside the container
        - mountPath: /opt
          name: data-volume
  volumes:
    - name: data-volume                   # Volume creation
      hostPath:                           # Directory on the host
        path: /data
        type: Directory
```

### Problem with Direct Volume Configuration
- Volume configuration is defined inside the Pod specification.
- In large environments, managing volumes for each Pod becomes difficult.
- Any change must be updated across all Pods manually.
  
---

### Persistent Volume

<p align="center">
  <img src="https://github.com/user-attachments/assets/cba32d85-dfbe-4f52-aa8c-f850b8d80de8" width="600"/>
</p>

A Persistent Volume (PV) is a cluster-wide pool of storage created by an administrator.
It can be used by applications deployed in the cluster and help manage storage centrally.
- Users do not directly use PVs.
- Instead, they request storage using Persistent Volume Claims (PVCs).

### PV Definition File:
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:                            # Access mode defines how the volume should be mounted on the host(ReadWriteOnce, ReadOnlyMany, ReadWriteMany)
    - ReadWriteOnce
  capacity:                               # Storage to be reserved for the volume
    storage: 1Gi
  awsElasticBlockStore:                   # Volume type used (host path is not an ideal choice in case of production)
    volumeID: <volume-id>                 # AWS elastic block storage is one of the supported storage solutions used here
    fsType: ext4
```
### Operational Commands
To create a volume:
```bash
kubectl create -f pv-definition.yaml
```
To list th PV:
```bash
kubectl get persistentvolume
```

---

### Persistent Volume Claim
- PV and PVC are separate objects in Kubernetes.
- The administrator creates PVs, and users create PVCs to request storage.
  
### Binding Process
Kubernetes binds a PVC to a suitable PV based on:
- Capacity
- Access modes
- Storage class
- Other properties
  
### Key points:
- Each PVC is bound to one PV.
- If multiple PVs match, selection can be controlled using labels and selectors.
- If no PV is available, the PVC remains in Pending state.
- A PVC can bind to a larger PV if no exact match exists.
- Once bound, no other PVC can use that PV.

### PVC Definition File:
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```
To create a persistent volume claim:
```bash
kubectl create -f pvc-definition.yaml
```
Deleting a PVC:
```bash
kubectl delete persistentvolumeclaim <pvc name>
```
- By default, PVs have a Retain policy.
- Deleting a PVC does not delete the PV.
- The PV remains but is not available for new claims.
- This behavior can be changed to Delete policy if required.

---

> Working flow of PV and PVC together:

<p align="center">
  <img src="https://github.com/user-attachments/assets/c3e0fcbf-0475-4e86-8ce9-df26a3821a68" width="600"/>
</p>

---

### Storage Class

<p align="center">
  <img src="https://github.com/user-attachments/assets/f95f7f87-1410-435d-990b-3621a316e755" width="600"/>
</p>

> Static Provisioning
- When creating a PV manually (e.g., using AWS EBS), you must first create the storage resource externally.
- Then define it in Kubernetes.
- This is called static provisioning.

> Dynamic Provisioning
- A StorageClass automates volume creation when required by an application.
- It defines a provisioner (e.g., AWS, GCP, Azure) to automate.

### StorageClass Definition File:
```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
  provisioner: kubernetes.io/gce-pd
```

### Using StorageClass in PVC
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage  
  resources:
    requests:
      storage: 500Mi
```
Once storage class is specified in PVC and PVC is created:
- StorageClass provisions storage automatically
- A PV is created dynamically
- PVC is bound to that PV

### Additional Parameters
StorageClass can include parameters specific to the provisioner:
```bash
parameters:
  type:               # Disk type (e.g., SSD, HDD)
  replication-type:   # Replication settings
```

> Storage classes can also represent tiers like:
Silver → Standard disks
Gold → SSD
Platinum → SSD with replication

<p align="center">
  <img src="https://github.com/user-attachments/assets/e473b462-2de1-48f9-bcf5-01cedd19f547" width="600"/>
</p>

---
