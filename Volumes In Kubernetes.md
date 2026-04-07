Volumes in Kuberentes

Docker Volumes
- containers are short lived. The data within the container is destroyed along with the container itself.
- Hence, to persist data with the container we use volumes. The data of the container is retained in the volume to persist the data permanently even if the container is deleted.

Volumes in Kubernetes
Similarly pods in kubernetes are transeint in nature, hence the data processed by pod is destroyed along with the pod.
To persist data even after the pod is destroyed, we attach a volume to the pod.
Once a volume is created we store it by providing a hostpath that is directory on a host.
Any data created in the volume is stored in the directory on the node
After the creation of volumes, to access it from within the container, we mount the volume to the directory withing the container.


Storing volume on the host path is not ideal choice in case of multiple node
Since all the nodes will be using the same path and will be expecting all nodes to have same data which is not the case.

Volume definition file example:
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
      volumeMounts:                       # to mount the volume within the container in order to access it from the container
        - mountPath: /opt
          name: data-volume
  volumes:
    - name: data-volume                   # creation of new volume
      hostPath:                           # host path to store a volume
        path: /data
        type: Directory

Volume configuration generally goes within the pod definition file.
But in case of large environment, it becomes hectic to configure volume on each pod.
Any change made, is required to be made on all ther pods everytime.


In order to manage storage more centrally , we make use of:
Persistent Volume:
<img width="900" height="510" alt="pv_diagram" src="https://github.com/user-attachments/assets/cba32d85-dfbe-4f52-aa8c-f850b8d80de8" />

A persistent Volume is a cluster wide pool of storage volumes, configured by an administrator to be used by user deployer application on a container.
The user can select storage from this pool using persistent volume claim.

pv definition file:
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


 To create a volume:
 kubectl create -f pv-definition.yaml

 To list th PV:
 kubectl get persistentvolume


 Persistent Volume Claims:
 PV and PVC are two seperate objects in kubernetes.
 An administrator created a persistent volume and a user creates a persistent volume claims to use the storage.

 Once the pvc are created, kubernetes binds the persistent volumes to claims based on the requests and properties set on the volumes.
 Every pvc is bound to single persistent volume.
 K8s make sures during binding process:
 - The persistent volume has sufficient capacity as requested by the pvc and any other requested properties like access modes, volume modes, storage class etc.
 In case there are multiple volumes that satisfies the same request, we can still bind to the volume of our choice using labels and selectors.

 - If no volumes are present, the pvc will remain in pending state until newer volume is available in the cluster.
 - A pvc can be attached to PV with greater capacity than needed if no other better matches exists and since pvc and pv has one to one relation, no other pvc can be attached.

PVC definition file:
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

To create a persistent volume claim:
kubectl create -f pvc-definition.yaml

<img width="1200" height="725" alt="pv_pvc_diagram" src="https://github.com/user-attachments/assets/c3e0fcbf-0475-4e86-8ce9-df26a3821a68" />


Deleting a PVC:
To delete a pvc:
kubectl delete persistentvolumeclaim <pvc name>

The underlying persistent volume is not deleted even if the pvc is destroyed, since by default pvs are set to retain policy and hence they exists until manually deleted.
However they are not available to use by any other claims.
They can be set to DELETE along with the PVCs.

Storage Class
<img width="1280" height="738" alt="dynamic_provisioning" src="https://github.com/user-attachments/assets/f95f7f87-1410-435d-990b-3621a316e755" />

when we create a pv
pv definition file:
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

We have used a aws elastic block storage here, hence we need to manually provision a storage block on aws before creating a pv with a same name in kubernetes.
This is called static provisioning.

Storage class help provisioning the volume automatically when the application needs it.
Storage class can help define a provisioner like aws, google storage etc. that can automatically provision storage on cloud and attach it to pod when claim is made.
This is called dynamic provisioning of volumes.

Storage class definition file:
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
  provisioner: kubernetes.io/gce-pd

Once we have a storage class created, we do not require a pv anymore because the pv and any other associated storage is going to be created automatically once the storage class is created.

For PVC to used storage class name, this is how we specify:
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

Once the storage class is defined in the pvc , the storage class uses the defined provisioner to provision a storage space on cloud and then creates a pv and bind it to the pvc.
PV is still created but this time we do not have to manually create it anymore.

Additionaly parametres can be passed with the type of storage class which is in use like
parameters:
  type:               # type of disk
  replication-type:   # replication enabled or disabled

Each parameters are specific to the provisioner used. 
We can also define the class of storage we want to use like silver class for standard disks, gold class for ssd drives and platinum class for ssd drives along with replication.
<img width="1442" height="596" alt="ChatGPT Image Apr 7, 2026, 03_13_11 PM" src="https://github.com/user-attachments/assets/e473b462-2de1-48f9-bcf5-01cedd19f547" />
