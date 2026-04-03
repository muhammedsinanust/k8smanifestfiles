# Dynamic Volume Provisioning

- Dynamic volume provisioning allows storage volumes to be created on-demand.
- Without dynamic provisioning, we have to manually create new storage volumes, and create.

PersistentVolume objects to represent them in Kubernetes.

- The dynamic provisioning feature eliminates the need to pre-provision storage.
- Automatically provisions storage when we create PersistentVolumeClaim objects.
The implementation of dynamic volume provisioning is based on the API object StorageClass from the API group storage.k8s.io.

## StorageClass

- Volume plugin (aka provisioner) that provisions a volume.
- Set of parameters to pass to that provisioner when provisioning.

A cluster administrator can define and expose multiple flavors of storage within a cluster, from the same or different storage systems, each with a custom set of parameters.
`it’s like a model that we can later specify the size and colour (classes and objects)`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: low-latency
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: csi-driver.example-vendor.example
reclaimPolicy: Retain # default value is Delete
allowVolumeExpansion: true
mountOptions:
  - discard # this might enable UNMAP / TRIM at the block storage layer
volumeBindingMode: WaitForFirstConsumer
parameters:
  guaranteedReadWriteLatency: "true" # provider-specific
```

If you set the `storageclass.kubernetes.io/is-default-class` annotation to `true` on more than one StorageClass in your cluster, and you then create a PersistentVolumeClaim with no storageClassName set, Kubernetes uses the most recently created default StorageClass.

> [!NOTE]
> Try to have only one default StorageClass in your cluster. Multiple default StorageClasses is to allow for seamless migration.

If you create a PersistentVolumeClaim without specifying a storageClassName, even when no default StorageClass exists. the new PVC creates as you defined it, and the storageClassName remains unset until a default becomes available.

when a default StorageClass becomes available, the control plane identifies existing PVCs without storageClassName. (empty value for storageClassName or do not have this key) the control plane then updates those PVCs to the new default StorageClass.
If you have an existing PVC where the storageClassName is "", and you configure a default StorageClass, then this PVC will not get updated.

## Provisioner

Each StorageClass has a provisioner that determines what volume plugin is used for provisioning PVs.

| Volume Plugin | Internal Provisioner | Config Example |
| --- | --- | --- |
| AzureFile | ✓ | Azure File |
| CephFS | - | - |
| FC | - | - |
| FlexVolume | - | - |
| iSCSI | - | - |
| Local | - | Local |
| NFS | - | NFS |
| PortworxVolume | ✓ | Portworx Volume |
| RBD | - | Ceph RBD |
| VsphereVolume | ✓ | vSphere |
