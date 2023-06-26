# **INSTALL CSI DRIVERS OFFLINE**

Deploy the CSI driver for the relevant Kubernetes version.

All components below are deployed in the "hpe-storage" `namespace`

```
    kubectl create namespace hpe-storage
```

### **Prerequisite before installing the CSI drivers**

Import or pull all the images which are listing in the below snippet using the `docker` or `podman` or `ctr`

```
alletra-6000-and-nimble-csp
csi-driver
csi-provisioner
csi-snapshotter
volume-group-snapshotter
csi-attacher
csi-extensions
csi-resizer
volume-group-provisioner
volume-mutator
csi-provisioner
```

Note: If any of the images are not available then the CSI drivers will failed to install.

### **Installation**

1. If the `hpe-linux-config.yaml` file is unavailable then download the file from the following link [hpe-linux-config.yaml](https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v2.2.0/hpe-linux-config.yaml), run the below command 

    > kubectl apply -f hpe-linux-config.yaml

2. a) If we are using the container storage provider is `HPE Alletra 6000 and Nimble Storage`,  then the `nimble-csp.yaml` and the link to download is [nimble-csp.yaml](https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v2.2.0/nimble-csp.yaml). 

    > kubectl apply -f nimble-csp.yaml

    b) In other option, if we are using the container storage provider is `HPE Alletra 9000, Primera and 3PAR`, then the files are `3par-primera-csp.yaml` and `3par-primera-crd.yaml` and the link to download is [3par-primera-csp.yaml](https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v2.2.0/3par-primera-csp.yaml) and [3par-primera-crd.yaml](https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v2.2.0/3par-primera-crd.yaml)

    
    > kubectl apply -f 3par-primera-csp.yaml \
    > kubectl apply -f 3par-primera-crd.yaml

3. To install csi driver with the version of kubernetes 1.24, download the link [hpe-csi-k8s-1.24.yaml](https://raw.githubusercontent.com/hpe-storage/co-deployments/master/yaml/csi-driver/v2.2.0/hpe-csi-k8s-1.24.yaml)

    > kubectl apply -f hpe-csi-k8s-1.24.yaml

### **Add an HPE Storage Backend**

Once the CSI driver is deployed, two additional objects needs to be created to get started with dynamic provisioning of persistent storage, a `Secret` and a `StorageClass`.

1. HPE Nimble Storage

```
apiVersion: v1
kind: Secret
metadata:
  name: hpe-backend
  namespace: hpe-storage
stringData:
  serviceName: nimble-csp-svc
  servicePort: "8080"
  backend: 192.168.1.2
  username: admin
  password: admin
```

Create the `Secret` using `kubectl`:

    kubectl create -f secret.yaml

To use the new Secret "custom-secret", create a new `StorageClass` using the `Secret` and the necessary `StorageClass` parameters. Please see the requirements section of the respective CSP.

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: hpe-scod
provisioner: csi.hpe.com
parameters:
  csi.storage.k8s.io/fstype: xfs
  csi.storage.k8s.io/controller-expand-secret-name: hpe-backend
  csi.storage.k8s.io/controller-expand-secret-namespace: hpe-storage
  csi.storage.k8s.io/controller-publish-secret-name: hpe-backend
  csi.storage.k8s.io/controller-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-publish-secret-name: hpe-backend
  csi.storage.k8s.io/node-publish-secret-namespace: hpe-storage
  csi.storage.k8s.io/node-stage-secret-name: hpe-backend
  csi.storage.k8s.io/node-stage-secret-namespace: hpe-storage
  csi.storage.k8s.io/provisioner-secret-name: hpe-backend
  csi.storage.k8s.io/provisioner-secret-namespace: hpe-storage
  description: "Volume created by the HPE CSI Driver for Kubernetes"
  accessProtocol: iscsi
reclaimPolicy: Delete
allowVolumeExpansion: true
```

    kubectl apply -f hpe-storageClas.yml


Create a `PersistentVolumeClaim`. This object declaration ensures a `PersistentVolume` is created and provisioned on your behalf, make sure to reference the correct .spec.storageClassName:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-first-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 32Gi
  storageClassName: hpe-scod
```

    kubectl apply -f hpe-pvc.yml


### **Get information**

```
kubectl get pods -n hpe-storage
kubectl get Deployment -n hpe-storage
kubectl get Deployment -n hpe-storage -o wide
kubectl get ServiceAccount -n hpe-storage
kubectl get ClusterRoleBinding -n hpe-storage
kubectl get Deployment -n hpe-storage -o wide
kubectl get pods -n hpe-storage
```