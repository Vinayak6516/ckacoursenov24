# 5. Configuring Pod Using HostPath-Based PV and PVC

## 5.1 Overview
In Kubernetes, a `HostPath` volume type allows you to mount a file or directory from the host node's filesystem into a Pod. While `HostPath` is useful for local storage requirements or accessing specific files on the node, it is typically not recommended for production use due to the lack of portability and scalability. However, it can be useful in development and testing environments. This tutorial demonstrates how to use a `HostPath` volume along with a Persistent Volume (PV) and Persistent Volume Claim (PVC) to mount storage into a Pod.

## 5.2 Concept
A `HostPath` volume in Kubernetes allows you to specify a directory or file on the node where your Kubernetes cluster is running. This volume can be used by Pods running on the same node to persist data. When using a `HostPath` volume in combination with a Persistent Volume and Persistent Volume Claim, you can dynamically manage storage and enable Pods to access this storage in a way that remains consistent even across restarts.

### How It Works:
- **HostPath PV**: A Persistent Volume (PV) is created that uses the `HostPath` volume type, linking it to a specific file or directory on the node's filesystem.
- **PVC**: A Persistent Volume Claim (PVC) is created by a Pod to request storage.
- **Pod**: The Pod is configured to use the PVC, thereby getting access to the data stored on the host machine.

## 5.3 Benefits
- **Cost-Efficient**: Useful for low-cost development environments where you need to access local storage.
- **Persistent Storage**: Provides persistent storage for Pods across restarts and rescheduling, even though the data is tied to the node.
- **Simplicity**: Easy to set up and configure for local storage needs.
- **Useful for Testing**: Can be used for testing scenarios where accessing local files on the node is required.

## 5.4 Use Cases
- **Local Development**: In development environments, where you want to use local disk storage rather than external cloud storage.
- **Node-Specific Data**: Storing data that is specific to a node and does not need to be replicated across nodes.
- **Testing and Debugging**: Useful in testing environments where quick and simple local storage is required.

## 5.5 Real-World Scenario
Imagine you have a Kubernetes cluster running an application that processes logs. These logs are saved locally on each node in a specific directory. To ensure that the application has access to the logs even if the Pod is rescheduled or restarted, you can use a `HostPath` Persistent Volume (PV) to link the Pod to this directory. The application can then continue reading and writing logs, even if it moves between nodes or restarts.

## 5.6 Implementation Example

### 5.6.1 Step 1: Create the HostPath Persistent Volume
First, we define a Persistent Volume (PV) that uses the `HostPath` volume type to point to a directory on the host.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  storageClassName: manual
  hostPath:
    path: /mnt/data
    type: Directory
```

### Explanation of the YAML:
- **hostPath.path**: Specifies the directory on the host node that will be used as the persistent volume.
- **accessModes**: `ReadWriteOnce` means the volume can be mounted by a single node for reading and writing.
- **persistentVolumeReclaimPolicy**: When the volume is released, it will not be deleted (set to `Retain`).

### 5.6.2 Step 2: Create the Persistent Volume Claim (PVC)
Next, we create a PVC that requests storage from the above-defined PV.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostpath-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Explanation of the YAML:
- **accessModes**: This matches the access mode of the PV (`ReadWriteOnce`).
- **resources.requests.storage**: The size of the volume requested from the PV, in this case, 1Gi.

### 5.6.3 Step 3: Create the Pod Using the PVC
Now, we define the Pod that uses the `HostPath` volume via the PVC.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: hostpath-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: hostpath-volume
    persistentVolumeClaim:
      claimName: hostpath-pvc
```

### Explanation of the YAML:
- **volumeMounts.mountPath**: Mounts the volume to the `/usr/share/nginx/html` directory in the container.
- **volumes.persistentVolumeClaim.claimName**: Refers to the PVC (`hostpath-pvc`), which requests the storage from the `HostPath` PV.

## 5.7 Verification Steps

1. **Create the Persistent Volume**:
   Apply the PV YAML to create the Persistent Volume:
   ```bash
   kubectl apply -f pv.yaml
   ```

2. **Create the Persistent Volume Claim**:
   Apply the PVC YAML to create the Persistent Volume Claim:
   ```bash
   kubectl apply -f pvc.yaml
   ```

3. **Create the Pod**:
   Apply the Pod YAML to create the Pod that uses the PVC:
   ```bash
   kubectl apply -f pod.yaml
   ```

4. **Verify the PVC is Bound to the PV**:
   Check if the PVC is correctly bound to the PV:
   ```bash
   kubectl get pvc hostpath-pvc
   ```

   The `STATUS` should be `Bound`.

5. **Check the Pod Status**:
   Check the status of the Pod to ensure it is running:
   ```bash
   kubectl get pods
   ```

6. **Verify the Mounted Volume**:
   Enter the Pod and check if the volume is correctly mounted to the specified path:
   ```bash
   kubectl exec -it hostpath-pod -- /bin/bash
   ls /usr/share/nginx/html
   ```

   You should see the content from the mounted directory (`/mnt/data` on the host node).