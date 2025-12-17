

# Single Data Center - L3LS - IPv6

## Introduction

This example is meant to be used as a second step in introducing AVD to new users, directly following the [Introduction to Ansible and AVD](../../../../../docs/getting-started/intro-to-ansible-and-avd.md) section. New users with access to virtual switches (here cEOS) can learn how to generate configuration and documentation for a complete fabric environment. Users with access to physical switches will have to adapt a few settings. 
This is all documented inline in the comments included in the YAML files.

The example includes and describes all the AVD files and their content used to build an IPv6 L3LS EVPN/VXLAN Symmetric IRB network covering a single DC using the following:

- Two (virtual) spine switches.
- Four sets of (virtual) leaf switches, serving endpoints such as servers.
- Four (virtual) layer2-only hosts, with connectivity in the different VLAN/VRF.

Ansible playbooks are included to show the following:

- Building the intended configuration and documentation
- Deploying the configuration directly to the switches using eAPI

## Installation

--8<--
ansible_collections/arista/avd/examples/common/example-installation.md
--8<--

```shell
single-dc-l3ls-ipv6
    ├── ansible.cfg
    ├── documentation
    ├── group_vars
    ├── host_vars
    ├── images
    ├── intended
    ├── inventory.yml
    ├── build.yml
    ├── deploy.yml
    ├── deploy-cvp.yml
    ├── README.md
    └── switch-basic-configurations
```

## Overall design overview

### Physical topology

The drawing below shows the physical topology used in this example. The interface assignment shown here are referenced across the entire example, so keep that in mind if this example must be adapted to a different topology. Finally, the Ansible host is the same running the Arista cEOS container nodes:

![Figure: Arista Leaf Spine physical topology](images/avd-ipv6-dc1.svg)`

### IP ranges used

| Out-of-band management IPv6 allocation for DC1        | 3fff:172:20:20::/64                       |
|-------------------------------------------------------|-------------------------------------------|
| Default gateway                                       | 3fff:172:20:20::1                         |
| spine01                                               | 3fff:172:20:20::201                       |
| spine02                                               | 3fff:172:20:20::202                       |
| leaf01                                                | 3fff:172:20:20::101                       |
| leaf02                                                | 3fff:172:20:20::102                       |
| leaf03                                                | 3fff:172:20:20::103                       |
| leaf04                                                | 3fff:172:20:20::104                       |
| leaf05                                                | 3fff:172:20:20::105                       |
| leaf06                                                | 3fff:172:20:20::106                       |
| leaf07                                                | 3fff:172:20:20::107                       |
| leaf08                                                | 3fff:172:20:20::108                       |
| host01                                                | 3fff:172:20:20::111                       |
| host02                                                | 3fff:172:20:20::112                       |
| host03                                                | 3fff:172:20:20::113                       |
| host04                                                | 3fff:172:20:20::114                       |
| **Point-to-point links between leaf and spine**       | **(Underlay)**                            |
| DC1                                                   | 2001:DB8:2::/48                           |
| **Loopback0 interfaces used for EVPN peering**        | 2001:DB8:1::/48                           |
| **Loopback1 interfaces used for VTEP**                | **(Leaf switches)**                       |
| DC1                                                   | 2001:DB8:5::/48                           |
| **VTEP Loopbacks used for diagnostics**               | **(Leaf switches)**                       |
| VRF10                                                 | 2602:0010:FF::/48                         |
| VRF11                                                 | 2602:0011:FF::/48                         |
| VRF12                                                 | 2602:0012:FF::/48]                        |
| **IPv6 SVIs (interface vlan...)**                     | **2001:DB8:`<VLAN-ID>`::1/48**            |
|                                                       | **2602:`<VRF-ID>`:FF:`<VLAN-ID>`::1/64**  |
| For example `interface VLAN11` has the IPv6 address:  | 2602:0010:FF:11::1/64                     |
| For example `interface VLAN21` has the IPv6 address:  | 2001:DB8:21::1/48                         |
| **MLAG Peer-link (interface vlan 4094)**              | **(Leaf switches)**                       |
| DC1                                                   | 2001:DB8:3::/48                           |
| **MLAG iBGP Peering (interface vlan 4093)**           | **(Leaf switches)**                       |
| DC1                                                   | 2001:DB8:4::/48                           |

### BGP design

=== "Underlay" (Based on physical interfaces)

![Figure: Arista Underlay BGP Design](images/AVD-IPv6-Underlay.svg)

=== "Overlay" (Based on logical "Loopback0" interfaces)

![Figure: Arista Underlay BGP Design](images/AVD-IPv6-Overlay.svg)

### EOS config 

The Arista cEOS nodes are configured to use the Ansible generated configuration.
!!! Note: Be sure to build them using "ansible-playbook build.yml" command prior to starting the simulation (check the "Playbook Run" section below).

## Ansible inventory, group vars, and naming scheme

The following drawing shows a graphic overview of the Ansible inventory, group variables, and naming scheme used in this example:

![Figure: Ansible inventory and vars](images/AVD-IPv6-DC1-Ansible-Groups.svg)

!!! note
    The 4 hosts `host0[1-4]` at the bottom are **also** configured by AVD, ans so are the switch ports used to connect the hosts to the FABRIC.

Group names use uppercase and underscore syntax:

- FABRIC
- DC1
- DC1_SPINES
- DC1_L3_LEAVES
- DC1_TENANT_HOSTS

All hostnames use lowercase:

- spine01
- leaf01
- leaf02

The drawing also shows the relationships between groups and their children:

- For example, `spine01` and `spine02` are both children of the group called `DC1_SPINES`.

Additionally, groups themselves can be children of another group, for example:

- `DC1_L3_LEAVES` is a group consisting of the groups `DC1_LEAF1`, `DC1_LEAF2`, `DC1_LEAF3` and `DC1_LEAF4`
- `DC1_L3_LEAVES` is also a child of the group `DC1`.

This naming convention makes it possible to extend anything easily, but as always, this can be changed based on your preferences. Just ensure that the names of all groups and hosts are unique.

### Content of the inventory.yml file

This section describes the entire `ansible-avd-examples/single-dc-l3ls-ipv6/inventory.yml` file used to represent the above topology.

It is important that the hostnames specified in the inventory exist either in DNS or in the hosts file on your Ansible host to allow successful name lookup and be able to reach the switches directly. A successful ping from the Ansible host to each inventory host verifies name resolution(e.g., `ping dc1-spine1`).

Alternatively, if there is no DNS available, or if devices need to be reached using a fully qualified domain name (FQDN), define `ansible_host` to be an IP address or FQDN for each device - see below for an example:

```yaml title="inventory.yml"
--8<--
inventory.yml
--8<--
```

The above is what is included in this example, *purely* to make it as simple as possible to get started. However, in the future, please do not carry over this practice to a production environment, where an inventory file for an identical topology should look as follows when using DNS:

```yaml title="inventory.yml"
--8<--
inventory_without_ip.yml
--8<--
```

1. `FABRIC`

   - FABRIC represents the highest level within the hierarchy.

   - Ansible variables defined at this level will be applied to all nodes in the fabric.

2. `NETWORK_SERVICES`

    - Creates a group named `NETWORK_SERVICES`. Ansible variable resolution resolves this group name to the identically named group_vars file (`group_vars/NETWORK_SERVICES.yml`).

    - The file's contents, which in this case are specifications of VRFs and VLANs, are then applied to the group's children. In this case, the two groups `DC1_L3_LEAVES` and `DC1_TENANT_HOSTS`.

3. `CONNECTED_ENDPOINTS`

    - Creates a group named `CONNECTED_ENDPOINTS`. Ansible variable resolution resolves this group name to the identically named group_vars file (`group_vars/CONNECTED_ENDPOINTS.yml`).

    - The file's contents, which in this case are specifications of connected endpoints (typically servers), are then applied to the children of the group, in this case, the two groups `DC1_L3_LEAVES` and `DC1_TENANT_HOSTS`.

## Defining device types

Since this example covers building an L3LS network, AVD must know about the device types, for example, spines, L3 leaves, L2 leaves, etc. The devices are already grouped in the inventory, so the device types are specified in the group variable files with the following names and content:

=== "spines.yml"

    ```yaml
    --8<--
    group_vars/DC1_SPINES/spines.yml
    --8<--
    ```

=== "l3_leaves.yml"

    ```yaml
    --8<--
    group_vars/DC1_L3_LEAVES/l3_leaves.yml
    --8<--
    ```

=== "tenant_hosts.yml"

    ```yaml
    --8<--
    group_vars/TENANT_HOSTS/tenant_hosts.yml
    --8<--
    ```

For example, all switches that are children of the DC1_SPINES group defined in the inventory will be of type `spine`.

## Setting fabric-wide configuration parameters

The `ansible-avd-examples/single-dc-l3ls-ipv6/group_vars/FABRIC` folder contain files that defines generic settings that apply to all children of the `FABRIC` group as specified in the inventory described earlier.

The first file defines how the Ansible host connects to the devices:

```yaml title="fabric_ansible_connectivity.yml"
--8<--
group_vars/FABRIC/fabric_ansible_connectivity.yml
--8<--
```

The following section specifies variables that generate configuration to be applied to all devices in the fabric:

```yaml title="fabric_variables.yml"
--8<--
group_vars/FABRIC/fabric_variables.yml
--8<--
```

## Setting device specific configuration parameters

The `ansible-avd-examples/single-dc-l3ls-ipv6/group_vars/DC1/dc1.yml` file defines settings that apply to all children of the `DC1` group as specified in the inventory described earlier. However, this time the settings defined are no longer fabric-wide but are limited to DC1. This example is of limited benefit with only a single data center. Still, it allows us to scale the configuration to a scenario with multiple data centers in the future.

```yaml title="dc1.yml"
--8<--
group_vars/DC1/dc1.yml
--8<--
```

The `ansible-avd-examples/single-dc-l3ls-ipv6/group_vars/DC1_SPINES/spines.yml` covers the spine switches.

```yaml title="spines.yml"
--8<--
group_vars/DC1_SPINES/spines.yml
--8<--
```

The `ansible-avd-examples/single-dc-l3ls-ipv6/group_vars/DC1_L3_LEAVES/l3_leaves.yml` covers the L3 leaf switches. Significantly more settings need to be set compared to the spine switches.

```yaml title="l3_leaves.yml"
--8<--
group_vars/DC1_L3_LEAVES/l3_leaves.yml
--8<--
```

Finally, for the Tenant Hosts:

```yaml title="tenant_hosts.yml"
--8<--
group_vars/DC1_TENANT_HOSTS/tenant_hosts.yml
--8<--
```

An L2 leaf/host switch is simpler than an L3 switch. Hence there are fewer settings to define.

## Specifying network services (VRFs and VLANs) in the EVPN/VXLAN fabric

The `ansible-avd-examples/single-dc-l3ls-ipv6/group_vars/NETWORK_SERVICES/network_services.yml` file defines All VRF and VLANs. This means that regardless of where a given VRF or VLAN must exist, its existence is defined in this file, but it does not indicate ***where*** in the fabric it exists. That was done at the bottom of the inventory file previously described in the [Inventory](#content-of-the-inventoryyml-file) section.

```yaml title="network_services.yml"
--8<--
group_vars/NETWORK_SERVICES/network_services.yml
--8<--
```

AVD offers granular control of where Tenants and VLANs are configured using `tags` and `filter`. Those areas are not covered in this basic example.

## Specifying endpoint connectivity in the EVPN/VXLAN fabric

After the previous section, all VRFs and VLANs across the fabric are now defined. The `ansible-avd-examples/single-dc-l3ls-ipv6/group_vars/CONNECTED_ENDPOINTS/connected_endpoints.yml` file specifies the connectivity for all endpoints in the fabric (typically servers):

```yaml title="connected_endpoints.yml"
--8<--
group_vars/CONNECTED_ENDPOINTS/connected_endpoints.yml
--8<--
```

## The playbooks

In this example, three playbooks are included, of which two must be used:

1. The first playbook `build.yml` is mandatory and is used to build the structured configuration, documentation and finally the actual EOS CLI configuration.
2. The second playbook`deploy.yml` to deploy the configurations generated by `build.yml` directly to the Arista switches using eAPI.
   
The `build.yml` playbook looks like the following:

```yaml title="build.yml"
--8<--
build.yml
--8<--
```

1. This sets the scope of the playbook, which in this example is the entire fabric. For instance, `FABRIC` is a group name defined in the inventory. If the playbook should only apply to a subset of devices, it can be changed here.
2. This task uses the role `arista.avd.eos_designs`, which generates structured configuration for each device. This structured configuration can be found in the `ansible-avd-examples/single-dc-l3ls-ipv6/intended/structured_configs` folder.
3. This task uses the role `arista.avd.eos_cli_config_gen`, which generates the Arista EOS CLI configurations found in the `ansible-avd-examples/single-dc-l3ls-ipv6/intended/configs` folder, along with the device-specific and fabric wide documentation found in the `ansible-avd-examples/single-dc-l3ls-ipv6/documentation/` folder. In addition, it relies on the structured configuration generated by `arista.avd.eos_designs`.

The `deploy.yml` playbook looks like the following:

```yaml title="deploy.yml"
--8<--
deploy.yml
--8<--
```

1. This sets the scope of the playbook, which in this example is the entire fabric. For instance, `FABRIC` is a group name defined in the inventory. If the playbook should only apply to a subset of devices, it can be changed here.
2. This task uses the `arista.avd.eos_config_deploy_eapi` role to deploy the configurations directly to EOS nodes that were generated by the `arista.avd.eos_cli_config_gen` role.

### Playbook Run

To build the configuration files, run the playbook called `build.yml`.

``` bash
### Build Configurations and Documentation
ansible-playbook playbooks/build.yml
```

After the playbook run finishes, EOS CLI intended configuration files were written to `intended/configs`.

To build and deploy the configurations to your switches directly, using eAPI, run the playbook called `deploy.yml`. This assumes that your Ansible host has access and authentication rights to the switches. Those auth variables are defined in FABRIC.yml.

``` bash
### Deploy Configurations to Devices using eAPI
ansible-playbook playbooks/deploy.yml
```

### EOS Intended Configurations

Your configuration files should be similar to these.

=== "spine1"

    ``` shell
    --8<--
    intended/configs/spine01.cfg
    --8<--
    ```

=== "spine2"

    ``` shell
    --8<--
    intended/configs/spine02.cfg
    --8<--
    ```

=== "leaf01"

    ``` shell
    --8<--
    intended/configs/leaf01.cfg
    --8<--
    ```

=== "leaf02"

    ``` shell
    --8<--
    intended/configs/leaf02.cfg
    --8<--
    ```

=== "leaf03"

    ``` shell
    --8<--
    intended/configs/leaf03.cfg
    --8<--
    ```

=== "leaf04"

    ``` shell
    --8<--
    intended/configs/leaf04.cfg
    --8<--
    ```

(and so on and so forth...)

The execution of the playbook should produce the following output:

```shell
user@ubuntu:~/ansible-avd-examples/single-dc-l3ls-ipv6$ ansible-playbook build.yml

PLAY [Build Configurations and Documentation] ************************************************************************************************

TASK [arista.avd.eos_designs : Verify Requirements] ******************************************************************************************
AVD version 5.7.2
Use -v for details.
ok: [spine01 -> localhost]

TASK [arista.avd.eos_designs : Create required output directories if not present] ************************************************************
ok: [spine01 -> localhost] => (item=/home/ubuntu/single-dc-l3ls-ipv6/intended/structured_configs)
ok: [spine01 -> localhost] => (item=/home/ubuntu/single-dc-l3ls-ipv6/documentation/fabric)

TASK [arista.avd.eos_designs : Set eos_designs facts] ****************************************************************************************
ok: [spine01]

TASK [arista.avd.eos_designs : Generate device configuration in structured format] ***********************************************************
ok: [spine01 -> localhost]
ok: [leaf01 -> localhost]
ok: [spine02 -> localhost]
ok: [leaf02 -> localhost]
ok: [leaf03 -> localhost]
ok: [leaf05 -> localhost]
ok: [leaf04 -> localhost]
ok: [leaf06 -> localhost]
ok: [leaf07 -> localhost]
ok: [leaf08 -> localhost]
ok: [host03 -> localhost]
ok: [host01 -> localhost]
ok: [host02 -> localhost]
ok: [host04 -> localhost]

TASK [arista.avd.eos_designs : Generate fabric documentation] ********************************************************************************
ok: [spine01 -> localhost]

TASK [arista.avd.eos_designs : Remove avd_switch_facts] **************************************************************************************
ok: [spine01]

TASK [arista.avd.eos_cli_config_gen : Verify Requirements] ***********************************************************************************
skipping: [spine01]

TASK [arista.avd.eos_cli_config_gen : Generate eos intended configuration and device documentation] ******************************************
ok: [spine01 -> localhost]
ok: [leaf02 -> localhost]
ok: [leaf03 -> localhost]
ok: [spine02 -> localhost]
ok: [leaf01 -> localhost]
ok: [leaf04 -> localhost]
ok: [leaf05 -> localhost]
ok: [leaf06 -> localhost]
ok: [leaf07 -> localhost]
ok: [leaf08 -> localhost]
ok: [host01 -> localhost]
ok: [host02 -> localhost]
ok: [host04 -> localhost]
ok: [host03 -> localhost]

PLAY RECAP ***********************************************************************************************************************************
host01                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
host02                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
host03                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
host04                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
leaf01                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
leaf02                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
leaf03                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
leaf04                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
leaf05                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
leaf06                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
leaf07                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
leaf08                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
spine01                    : ok=7    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
spine02                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
(...)
```

If similar output is not shown, make sure:

1. The documented [requirements](https://github.com/aristanetworks/avd/blob/devel/docs/installation/collection-installation.md) are met.
2. The latest `arista.avd` collection is installed.

## Troubleshooting

### EVPN not working

If after doing the following steps:

1. Manually copy/paste the switch-basic-configuration to the devices.
2. Run the playbook and push the generated configuration to the fabric.
3. Log in to a leaf device, for example, leaf01 and run the command `show bgp evpn summary` to view EVPN routes.

The following error message is shown:

```eos
leaf01#show bgp evpn summ
% Not supported
leaf01#
```

This is caused by AVD pushing the configuration line `service routing protocols model multi-agent`, which enables the multi-agent routing process supporting EVPN. This change *requires* a reboot of the device.
