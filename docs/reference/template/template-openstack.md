# OpenStack Machine parameters

## ClusterDeployment Parameters

To deploy an OpenStack cluster, the following are the primary parameters in the `ClusterDeployment` resource:

| Parameter                         | Example                               | Description                                                 |
|-----------------------------------|---------------------------------------|-------------------------------------------------------------|
| `.spec.credential`                | `openstack-cluster-identity-cred`     | Reference to the `Credential` object.                       |
| `.spec.template`                  | `openstack-standalone-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.openstackStandaloneCpCluster }}}` | Reference to the `ClusterTemplate`. |
| `.spec.config.authURL`            | `https://keystone.yourorg.net/`       | Keystone authentication endpoint for OpenStack.             |
| `.spec.config.controlPlaneNumber` | `3`                                   | Number of control plane nodes.                              |
| `.spec.config.workersNumber`      | `2`                                   | Number of worker nodes.                                     |
| `.spec.config.clusterLabels`      | `k0rdent: demo`                       | Labels to apply to the cluster. Used by MultiClusterService.|
| `.spec.config.ccmRegional`         | `true`                                | Enables the OpenStack CCM OS_CCM_REGIONAL envvar feature and allows OpenStack CCM to define the region in nodes |

### SSH Configuration

To access deployed machines over ssh requires two things:

- `sshKeyName` - the reference name for an existing SSH key configured in OpenStack.
- `bastion` - bastion host being enabled and its flavor and image specified.

#### SSH keys

Specify the SSH public key using the `.spec.config.controlPlane.sshKeyName` and `.spec.config.worker.sshKeyName` parameters (for the standalone control plane).

#### Bastion

Specify `.spec.config.bastion.enabled` to enable it as well as provide `sshKeyName`, `flavor` and `image` in `.spec.config.bastion.spec`, similarly to workers and control plane.

Example `ClusterDeployment with enabled bastion can be found [below](#example-clusterdeployment).

### Machine Configuration

Configurations for control plane and worker nodes are specified separately under `.spec.config.controlPlane` and `.spec.config.worker`:

| Parameter                      | Example               | Description                             |
|--------------------------------|-----------------------|-----------------------------------------|
| `flavor`                       | `m1.medium`           | OpenStack flavor for the instance.      |
| `image.filter.name`            | `ubuntu-22.04-x86_64` | Name of the image.                      |
| `sshKeyName`                   | `ramesses-pk`         | Reference name for an existing SSH key. |
| `securityGroups[].filter.name` | `default`             | Security group for the instance.        |

> NOTE:
> Make sure `.spec.credential` references the `Credential` object.
> The recommended minimum vCPU value for the control plane flavor is 2, while for the worker node flavor, it is 1. For detailed information, refer to the [machine-flavor CAPI docs](https://github.com/kubernetes-sigs/cluster-api-provider-openstack/blob/main/docs/book/src/clusteropenstack/configuration.md#machine-flavor).

### External Network Configuration

If your OpenStack cloud contains more than one network marked as external it is necessary to provide which one clusterapi should use when creating a cluster. You do this by providing `.spec.config.externlNetwork.filter.name` value with the name of your external network.

### Load Balancer Configuration

If your user doesn't have access to or your cloud doesn't utilize octavia load balancer it is possible to disable usage of it by specifying
`.spec.config.apiServerLoadBalancer.enabled` as `false`.

> WARNING: Disabling loadbalancer blocks usage of `LoadBalancer` type services in cluster until one is manually installed.

### Configuring some of k0s, k0smotron parameters

- `k0s.arch` (string): Defines K0s Arch in its download URL. Available if [global.k0sURL](../../appendix/appendix-extend-mgmt.md#configuring-a-global-k0s-url)
   is set. Possible values: `"amd64"` (default), `"arm64"`, `"arm"`.
- `k0s.cpArgs` (array of strings): A list of extra arguments to be passed to k0s controller.
   See: <https://docs.k0sproject.io/stable/cli/k0s_controller>.
- `k0s.workerArgs` (array of strings): A list of extra arguments for configuring the k0s worker node. See: <https://docs.k0sproject.io/stable/cli/k0s_worker>.

### Example ClusterDeployment

```yaml
apiVersion: k0rdent.mirantis.com/v1beta1
kind: ClusterDeployment
metadata:
  name: my-openstack-cluster-deployment
  namespace: kcm-system
spec:
  template: openstack-standalone-cp-{{{ extra.docsVersionInfo.providerVersions.dashVersions.openstackStandaloneCpCluster }}}
  credential: openstack-cluster-identity-cred
  config:
    clusterLabels:
      k0rdent: demo
    controlPlaneNumber: 1
    workersNumber: 1
    bastion:
      enabled: true
      spec:
        sshKeyName: my-public-key
        flavor: m1.small
        image:
          filter:
            name: ubuntu-22.04-x86_64
    controlPlane:
      sshKeyName: bastion-public-key
      flavor: m1.medium
      image:
        filter:
          name: ubuntu-22.04-x86_64
    worker:
      sshKeyName: bastion-public-key
      flavor: m1.medium
      image:
        filter:
          name: ubuntu-22.04-x86_64
    externalNetwork:
      filter:
        name: "public"
    authURL: https://my-keystone-openstack-url.com
    identityRef:
      name: openstack-cloud-config
      cloudName: openstack
      region: RegionOne
```
