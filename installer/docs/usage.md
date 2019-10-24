## Usage

### Configuration

See [full configuration](configuration.md) for all possible configuration
options.

### Patching

`installer` supports patching/overriding various portions of the generated
configuration, by convention it requires an `overlays` directory with one or
more patch files. The files need to contain at minimum the `apiVersion` `kind`
and `name` (`metadata.name`) of the object to be patched. `installer` will
do a
[`strategicmergepatch`](https://kubernetes.io/docs/tasks/run-application/update-api-object-kubectl-patch/#use-a-strategic-merge-patch-to-update-a-deployment)
of the patch file on the object. See
[examples/overlays_example](examples/overlays_example) for an example of how
to patch. Also see
[overlays.md](overlays.md) for a list of all `apiVersion` `kind` and `name`
that can be patched over.

NB: Patching should be used sparingly, if there are cases where it is used an
issue should be opened on this repo so we can explore ways to bake the use
cases into installer directly.

### Non-Airgap deployment

This assumes the Kubernetes cluster has network access to pulling images from
quay.io.

NB: installer will generate
[imagePullSecrets](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod),
so you do not have to worry about authenticated access to quay.io.

#### Requirements for installation machine

- Network access to Kubernetes cluster
- Docker
- Edited [sysdig-chart/values.yaml](sysdig-chart/values.yaml)

#### Workflow

- Login to quay.io see [Logging in to quay.io](../README.md#logging-in-to-quayio-io)
- Copy [sysdig-chart/values.yaml](sysdig-chart/values.yaml) to your
working directory, you can do:
```bash
wget \
https://raw.githubusercontent.com/draios/installer/2.5.0-1/sysdig-chart/values.yaml
```
- Modify the values.yaml
- Run
```bash
docker run --user=$(id -u) -e KUBECONFIG=/.kube/config -v ~/.kube:/.kube:Z \
  -v $(pwd):/manifests:Z quay.io/sysdig/installer:2.5.0-1
```

### Airgap installation with installation machine
[multi-homed](https://en.wikipedia.org/wiki/Multihoming)

This assumes a private docker registry is used and the installation machine
has network access to pull from quay.io and push images to the private
registry.

#### Requirements for installation machine

- Network access to Kubernetes cluster
- Docker
- Bash
- [jq](https://stedolan.github.io/jq/)
- Network access to quay.io
- Network and authenticated access to the private registry
- Edited [sysdig-chart/values.yaml](sysdig-chart/values.yaml), with airgap
registry details updated

#### Workflow

- Login to quay.io see [Logging in to quay.io](../README.md#logging-in-to-quayio-io)
- Copy [sysdig-chart/values.yaml](sysdig-chart/values.yaml) to your
working directory, you can do:
```bash
wget \
https://raw.githubusercontent.com/draios/installer/2.5.0-1/sysdig-chart/values.yaml
```
- Modify the values.yaml
- Run
```bash
docker run -e HOST_USER=$(id -u) -e KUBECONFIG=/.kube/config \
  -v ~/.kube:/.kube:Z \
  -v $(pwd):/manifests:Z \
  -v /var/run/docker.sock:/var/run/docker.sock:Z \
  -v ~/.docker:/root/docker:Z \
  quay.io/sysdig/installer:2.5.0-1
```

### Full Airgap installation

This assumes a private docker registry is used and the installation machine
does not have network access to pull from quay.io, but can push images to the
private registry. In this situation a machine with network access will be used
to pull an image that contains self extracting tarball that can be copied to
the installation machine, we will call this machine jump machine.

#### Requirements for jump machine

- Network access to quay.io
- Docker
- [jq](https://stedolan.github.io/jq/)

#### Requirements for installation machine

- Network access to Kubernetes cluster
- Docker
- Bash
- [tar](https://linux.die.net/man/1/tar)
- Network and authenticated access to the private registry
- Edited [sysdig-chart/values.yaml](sysdig-chart/values.yaml), with airgap
registry details updated

#### Workflow

**On Jump Machine**
- Login to quay.io
  - Retrieve quay username and password from quaypullsecret, e.g:
  ```bash
  AUTH=$(echo <REPLACE_WITH_quaypullsecret> | base64 -d | jq -r '.auths."quay.io".auth'| base64 -d)
  QUAY_USERNAME=${AUTH%:*}
  QUAY_PASSWORD=${AUTH#*:}
  ```
  - Use QUAY_USERNAME and QUAY_PASSWORD retrieved from previous step to login
  to quay
  ```bash
  docker login -u "$QUAY_USERNAME" -p "$QUAY_PASSWORD" quay.io
  ```
- Pull image containing self-extracting tar:
```bash
docker pull quay.io/sysdig/installer:2.5.0-1-uber
```
- Extract the tarball:
```bash
docker create --name uber_image quay.io/sysdig/installer:2.5.0-1-uber
docker cp uber_image:/sysdig_installer.tar.gz .
docker rm uber_image
```
- Copy the tarball to the installation machine

**On the installation machine**

- Copy [sysdig-chart/values.yaml](sysdig-chart/values.yaml) to your
working directory, you can do:
```bash
wget \
https://raw.githubusercontent.com/draios/installer/2.5.0-1/sysdig-chart/values.yaml
```
- Modify the values.yaml
- Copy the tar file to the directory
- Run the tar file `bash sysdig_installer.tar.gz`

### `hostPath` Storage

To make installer set up the Sysdig Platform correctly when `hostPath`
[`storageClassProvisioner`](../configuration.md#storageclassprovisioner) is
selected, the below is expected to be configured:

- [`storageClassProvisioner`](../configuration.md#storageclassprovisioner)
configured as `hostPath`.
- [`sysdig.cassandra.hostPathNodes`](../configuration.md#sysdigcassandrahostpathnodes),
the number of nodes configured here needs to be at minimum
[`sysdig.cassandraReplicaCount`](../configuration#sysdigcassandrareplicacount).
- [`elasticsearch.hostPathNodes`](../configuration.md#elasticsearchhostpathnodes),
the number of nodes configured here needs to be at minimum
[`elasticsearchReplicaCount`](../configuration#elasticsearchreplicacount).
- [`sysdig.mysql.hostPathNodes`](../configuration.md#sysdigmysqlhostpathnodes),
when [`sysdig.mysqlHa`](../configuration.md##sysdigmysqlha) is configured to
`true` this has to be at least 3 nodes and when
[`sysdig.mysqlHa`](../configuration.md##sysdigmysqlha) is not configured it
should be at least one node.
- [`sysdig.postgresql.hostPathNodes`](../configuration.md#sysdigpostgresqlhostpathnodes),
this can be ignored if [`apps`](../configuration.md#apps) is configured to
`monitor`.

An example entry in values.yaml will look like below:

```yaml
storageClassProvisioner: hostPath
elasticsearch:
  hostPathNodes:
    - my-cool-host1.com
    - my-cool-host2.com
    - my-cool-host3.com
    - my-cool-host4.com
    - my-cool-host5.com
    - my-cool-host6.com
sysdig:
  cassandra:
    hostPathNodes:
      - my-cool-host1.com
      - my-cool-host2.com
      - my-cool-host3.com
      - my-cool-host4.com
      - my-cool-host5.com
      - my-cool-host6.com
  mysql:
    hostPathNodes:
      - my-cool-host1.com
  postgresql:
    hostPathNodes:
      - my-cool-host1.com
```

With the above installer will create [Persistent
Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
bound to the configured nodes and [Persistent Volume
Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
for that the various pods will use to bind the Persistent Volumes. To override
the prefix for the path on the filesystem where the various hostPath volumes
are bound to, see
[`hostPathCustomPaths.cassandra`](../configuration#hostpathcustompathscassandra),
[`hostPathCustomPaths.elasticsearch`](../configuration#hostpathcustompathselasticsearch),
[`hostPathCustomPaths.mysql`](../configuration#hostpathcustompathsmysql),
[`hostPathCustomPaths.postgresql`](../configuration#hostpathcustompathspostgresql).


### Local Storage

When `local` storage is selected, installer uses
https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner to
manage creation of persistent volumes. Requirement for this to work is that
volumes are created and mounted under the directory `/sysdig`, e.g:

```bash
tree /sysdig/
/sysdig/
├── vol1
├── vol2
└── vol3
```

on every node in the cluster. Below is an example of creating such volumes
using a [`loop device`](https://en.wikipedia.org/wiki/Loop_device):

```bash
parallel-ssh -t 0 --inline-stdout -l admin -x "-o StrictHostKeyChecking=no \
  -o UserKnownHostsFile=/dev/null -o GlobalKnownHostsFile=/dev/null -t" -h \
  <(kubectl get nodes --selector='!node-role.kubernetes.io/master' -o json | \
  jq -r '.items[].status.addresses[] | select (.type == \
  "ExternalIP").address') \
  "sudo bash -c ' \
    for i in \$(seq 1 3); do \
      mkdir -p /sysdig/vol\${i}; \
      dd if=/dev/zero of=/vol\${i} bs=1024 count=35000000; \
      mkfs.ext4 /vol\${i}; \
      mount /vol\${i} /sysdig/vol\${i}; \
    done \
  '"
```

The above creates 3 35G volumes per Kubernetes node. _Use the above as a last
resort and prefer creating raw volumes_. Volume requirements for a default
setup are as below:

Cluster size | Minimum Volume Size | Minimum number of volumes|
|-----|----|---|
small | 35G | 4|
medium | 110G | 8|
large | 320G | 14|

Minimum number of volumes is determined by `cassandraReplicaCount` +
`elasticSearchReplicaCount` + 2. If those are configured, ensure the number of
created volumes is greater than or equal to the new sum.

