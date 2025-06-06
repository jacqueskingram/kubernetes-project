# How to Use it?

NOTE: Ceph RBD in-tree storage driver ( `kubernetes.io/rbd`) has been
deprecated in kubernetes 1.28 release and will be in removed in subsequent
releases. The CSI driver for Ceph RBD (https://github.com/ceph/ceph-csi)
is available for long time now which can be an alternative option for the
users want to use Ceph RBD volumes in their clusters.

Install Ceph on the Kubernetes host. For example, on Fedora 21

    # yum -y install ceph-common

If you don't have a Ceph cluster, you can set up a [containerized Ceph cluster](https://github.com/ceph/ceph-docker)

Then get the keyring from the Ceph cluster and copy it to */etc/ceph/keyring*.

Once you have installed Ceph and new Kubernetes, you can create a pod based on my examples [rbd.yaml](rbd.yaml)  [rbd-with-secret.yaml](rbd-with-secret.yaml). In the pod JSON, you need to provide the following information.

- *monitors*:  Ceph monitors.
- *pool*: The name of the RADOS pool, if not provided, default *rbd* pool is used.
- *image*: The image name that rbd has created.
- *user*: The RADOS user name. If not provided, default *admin* is used.
- *keyring*: The path to the keyring file. If not provided, default */etc/ceph/keyring* is used.
- *secretName*: The name of the authentication secrets. If provided, *secretName* overrides *keyring*. Note, see below about how to create a secret.
- *fsType*: The filesystem type (ext4, xfs, etc) that formatted on the device.
- *readOnly*: Whether the filesystem is used as readOnly.

# Use Ceph Authentication Secret

If Ceph authentication secret is provided, the secret should be first be *base64 encoded*, then encoded string is placed in a secret yaml. For example, getting Ceph user `kube`'s base64 encoded secret can use the following command:

```console
  # grep key /etc/ceph/ceph.client.kube.keyring |awk '{printf "%s", $NF}'|base64
QVFBTWdYaFZ3QkNlRGhBQTlubFBhRnlmVVNhdEdENGRyRldEdlE9PQ==
```

An example yaml is provided [here](secret/ceph-secret.yaml). Then post the secret through ```kubectl``` in the following command.

```console
    # kubectl create -f examples/volumes/rbd/secret/ceph-secret.yaml
```

# Get started

Here are my commands:

```console
    # kubectl create -f examples/volumes/rbd/rbd.yaml
    # kubectl get pods
```

On the Kubernetes host, I got these in mount output

```console
    #mount |grep kub
	/dev/rbd0 on /var/lib/kubelet/plugins/kubernetes.io/rbd/rbd/kube-image-foo type ext4 (ro,relatime,stripe=4096,data=ordered)
	/dev/rbd0 on /var/lib/kubelet/pods/ec2166b4-de07-11e4-aaf5-d4bed9b39058/volumes/kubernetes.io~rbd/rbdpd type ext4 (ro,relatime,stripe=4096,data=ordered)
```

 If you ssh to that machine, you can run `docker ps` to see the actual pod and `docker inspect` to see the volumes used by the container.

