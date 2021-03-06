[![Validation CI](https://github.com/ansible-content-lab/lab.aws_roles/actions/workflows/validate.yml/badge.svg)](https://github.com/ansible-content-lab/lab.aws_roles/actions/workflows/validate.yml)

# Ansible Collection - lab.aws_roles

This repository hosts the `lab.aws_roles` Ansible Collection.

The collection includes a variety of Ansible roles and playbook to help automate the management of resources on AWS.

This content was developed as part of the [Ansible Content Lab for Cloud Content](https://ansible-content-lab.github.io/), a program from the Ansible team to help incubate Ansible cloud use cases from ideation to collections and roles.

## Included Content

<!--start collection content-->
### Roles

Click on the role name to be directed to the README specifically for that role.

| Name                                                                                                                                                          | Description                                                                                                                               |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| [lab.aws_roles.manage_direct_peered_networks](https://github.com/ansible-content-lab/lab.aws_roles/blob/main/roles/manage_direct_peered_networks/README.md)   | A role to create a VPC networking architecture that includes peer networking DMZ and private networks.                                    |
| [lab.aws_roles.manage_transit_peered_networks](https://github.com/ansible-content-lab/lab.aws_roles/blob/main/roles/manage_transit_peered_networks/README.md) | A role to create a hub-and-spoke VPC networking architecture that includes DMZ and private networks.                                      |
| [lab.aws_roles.peer_existing_networks](https://github.com/ansible-content-lab/lab.aws_roles/blob/main/roles/peer_existing_networks/README.md)                 | A role to automate the peering of two or more VPCs through direct peering model.                                                          |
| [lab.aws_roles.peer_transit_network](https://github.com/ansible-content-lab/lab.aws_roles/blob/main/roles/peer_transit_network/README.md)                     | Peers a VPC into an AWS transit gateway and configures routing rules across all attached VPCs to allow traffic to the newly attached VPC. |

### Playbooks

| Name                                    | Role(s) Used                           | Description                                                                                                                 |
|-----------------------------------------|----------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| `lab.aws_roles.create_peer_network`     | `roles.manage_direct_peered_networks`  | A playbook to create a multi-VPC peer network configuration with DMZ and private networks.                                  |
| `lab.aws_roles.delete_peer_network`     | `roles.manage_direct_peered_networks`  | Deletes AWS resources created in the `create_peer_network` playbook.                                                        |
| `lab.aws_roles.peer_networks`           | `roles.peer_networks`                  | Peer two or more VPCs with VPC peering.                                                                                     |
| `lab.aws_roles.create_transit_network`  | `roles.manage_transit_peered_networks` | A playbook to create a multi-VPC hub-and-spoke network configuration using a transit gateway with DMZ and private networks. |
| `lab.aws_roles.delete_transit_network`  | `roles.manage_transit_peered_networks` | Deletes AWS resources created in the `create_transit_network` playbook.                                                     |
| `lab.aws_roles.peer_to_transit_network` | `roles.peer_transit_network`           | A playbook to execute the Transit Gateway peering operation in the role used.                                               |
| `lab.aws_roles.create_vm`               | N/A                                    | Simple playbook to create an AWS VM.                                                                                        |
| `lab.aws_roles.delete_vm`               | N/A                                    | Deletes the VM created in the `create_vm` playbook.                                                                         |
<!--end collection content-->

#### VM Playbooks

The `lab.aws_roles.create_vm` and `lab.aws_roles.delete_vm` playbooks demonstrate how you can construct automation to deploy AWS resources that have dependencies on others.  The more complex networking roles and playbooks automate building all of the resources, but these playbooks assume that you have existing infrastructure (VPCs, security groups, SSH keys, etc.) that you want to leverage to deploy the EC2 instance.  The command below shows how you can use Ansible Navigator to deploy the instance with variables being set both in var files and directly in the CLI.

```yaml
ansible-navigator run playbooks/create_vm.yml \
--pae false \
--mode stdout \
--ee true \
--eei quay.io/scottharwell/cloud-ee \
--extra-vars "@playbooks/vars/create_vm.yml" \
--extra-vars "vpc_subnet_id=subnet-0e5c2afbb..." \
--extra-vars "security_group_id=sg-08814ac6..." \
--extra-vars "ssh_key_name=my_key" \
--eev $HOME/.ssh:/home/runner/.ssh \
--penv AWS_ACCESS_KEY \
--penv AWS_SECRET_ACCESS_KEY
```

#### Create Network Playbooks

The `lab.aws_roles.create_peer_network` and `lab.aws_roles.create_transit_network` playbooks have another tasks block that will attempt to configure the EC2 resources deployed by the roles a bit farther.  When the role completes, EC2 instances in the DMZ will still need to be configured with SSH configuration in order to communicate with EC2 instances in the private network(s).

To connect to the DMZ EC2 instance, the `ansible_ssh_private_key_file` variable needs to be set so that the machine running the playbook can connect to the newly created EC2 instance.  You may set this variable in any way that Ansible allows, i.e. extra var, host var, etc.  It must be set or the configuration step will be skipped.  The `ansible_ssh_user` variable is set automatically to the user `ec2-user` that is standard on AWS AMIs.

When the following variables are set, then the playbook will also configure the DMZ EC2 instance with the SSH config and a private key to communicate with each other.  This leaves the deployment in an immediately accessible state.

| Variable                                  | Use                                                                                                                |
|-------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| `priv_network_ssh_key_name`               | The AWS key name used for configuring SSH access to EC2 instances on the private VPC deployed in these roles.      |
| `ansible_ssh_private_key_file_local_path` | The local path to an SSH private key that will be copied to the bastion host.                                      |
| `ansible_ssh_private_key_file_dest_path`  | The destination path for the SSH key on the bastion host.                                                          |
| `priv_network_hosts_pattern`              | A hosts pattern that will be added to the bastion host SSH config for easier access to machines on other networks. |
| `priv_network_ssh_user`                   | The SSH user that will be configured in the bastion host SSH config.                                               |

```yaml
# These variables determine if there is an attempt to configure the DMZ VM to connect to other VMs.
priv_network_ssh_key_name: aws-test-key # The name of the AWS SSH key used to configure EC2 instances on the private network
ansible_ssh_private_key_file_local_path: ~/.ssh/aws-test-key.pem # Must exist locally or be mapped in an EE
ansible_ssh_private_key_file_dest_path: ~/.ssh/aws-test-key.pem # This will be the destination for the private key
priv_network_hosts_pattern: 10.* # Will use the same username and ssh key for any host on the 10.* network
priv_network_ssh_user: ec2-user # Will likely always be `ec2-user`, but set here as an option
```

## Installation and Usage

### Installing the Collection with the Ansible Galaxy CLI

Before using the this collection, you need to install it with the Ansible Galaxy CLI:

`ansible-galaxy collection install git+https://github.com/ansible-content-lab/lab.aws_roles.git`

You can also include it in a `requirements.yml` file and install it via `ansible-galaxy collection install -r requirements.yml`, using the format:

```yaml
---
collections:
  - name: https://github.com/ansible-content-lab/lab.aws_roles.git
    type: git
    version: main
```

### Adding a Git Pre-Commit Hook

This repo includes a configuration file for `ansible-lint` to be run as a git [pre-commit](https://pre-commit.com/) hook. To install the hook, run `pre-commit install` from the root directory of this repo once you have the pre-commit utility installed on your machine.

# License
GNU General Public License v3.0 or later

See [LICENCE](https://github.com/ansible-content-lab/lab.aws_roles/blob/main/LICENSE) to see the full text.