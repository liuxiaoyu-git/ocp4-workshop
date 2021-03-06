# Machine Config Operator (MCO)

Manage configuration changes and updates to essentially everything between the kernel and kubelet for different machine pools (worker, master, infra, etc).

## Machine config (MC)

The machines using RHCOS will be configured using an Ignition config served to the machines at first boot. An in-cluster Ignition endpoint will serve these Ignition configs to new machines based on the MachineConfig object defined.

### Modify NTP server

- **Master**: Yes
- **Worker**: Yes

Create a machine-config to overwrite `/etc/chrony.conf` in each node with the following configuration.

```
server clock.redhat.com iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```

## Machine config pool (MCP)

A MCP renders all machine config that match the `machineConfigSelector` expression and applies the change to all nodes that match the `nodeSelector` expression.

### Infra

#### machineConfigSelector

```yaml
matchExpressions:
  - values:
      - worker
      - infra
    operator: In
    key: machineconfiguration.openshift.io/role
```

#### nodeSelector

```yaml
matchLabels:
  node-role.kubernetes.io/infra: ""
```

## Scheduler configuration

The default scheduler can be configured to allocate pods always in worker nodes to avoid running applications in infra nodes.

```yaml
apiVersion: config.openshift.io/v1
kind: Scheduler
metadata:
  name: cluster
spec:
  defaultNodeSelector: node-role.kubernetes.io/worker=
  mastersSchedulable: false
  policy:
    name: ""
status: {}
```

**IMPORTANT**: This is not well tested and many operators failed to use default the node selector. Use this property at namespace label.

## Upgrade path

Select the channel and the architecture to get the upgrade path.

```bash
OCP_CHANNEL="stable-4.3"
OCP_ARCHITECTURE="amd64"
```

Run the following command to generate the upgrade path graph and open the `SVG` file with your favourite image viewer.

```bash
curl -sH "Accept:application/json" "https://api.openshift.com/api/upgrades_info/v1/graph?channel=${OCP_CHANNEL}&arch=${OCP_ARCHITECTURE}" | upgrade/graph.sh | dot -Tsvg > upgrade/graph-${OCP_CHANNEL}.svg
```

## Configuration

Apply the changes given by the previous configuration.

```bash
./install.sh
```

## References

- https://github.com/openshift/machine-config-operator