# Build a Kubernetes cluster using k3s via Ansible

Author: <https://github.com/geoff-coppertop>

## K3s Ansible Playbook

Build a Kubernetes cluster using Ansible with k3s. The goal is easily install a Kubernetes cluster on machines running:

- [X] Debian
- [X] Ubuntu
- [X] CentOS

on processor architecture:

- [X] x64
- [X] arm64
- [X] armhf

## System requirements

Deployment environment must have Ansible 2.4.0+
Master and nodes must have passwordless SSH access

## Playbook Usage

First create a new directory based on the `sample` directory within the `inventory` directory:

```bash
cp -R inventory/sample inventory/my-cluster
```

Second, edit `inventory/my-cluster/hosts.ini` to match the system information gathered above. For example:

```ini
[master]
192.16.35.12

[node]
192.16.35.[10:11]

[k3s_cluster:children]
master
node
```

If needed, you can also edit `inventory/my-cluster/group_vars/all.yml` to match your environment.

Start provisioning of the cluster using the following command:

```bash
ansible-playbook playbooks/site.yml -i inventory/my-cluster/hosts.ini
```

The cluster can be reset to its previous un-provisioned state using the following command:

```bash
ansible-playbook playbooks/reset.yml -i inventory/my-cluster/hosts.ini
```

## Ansible-Galaxy

### Publishing k3s roles collection

The roles are automatically pushed to [Ansible Galaxy](https://galaxy.ansible.com) via the [artis3n/ansible_galaxy_collection](https://github.com/artis3n/ansible_galaxy_collection) github action when the project is tagged v#.#.#. The publish action requires that an environment variable `ANSIBLE_GALAXY_TOKEN` to be set with API key for the account/namespace in question at [Ansible Galaxy](https://galaxy.ansible.com).

### Using k3s roles collection

The published collection can be used by first installing it using either,

```bash
ansible-galaxy collection install <namespace>.k3s_roles
```

Where `<namespace>` is replaced with the one used in the publishing step (e.g. geoff_coppertop). Note, this can also be accomplished with a requirements.yml file.

The collection can then be used in a playbook as follows,

```yml
- hosts: cluster
  become: yes
  gather_facts: yes
  roles:
  - geoff_coppertop.k3s_roles.prereq
  - geoff_coppertop.k3s_roles.download
  - geoff_coppertop.k3s_roles.raspberrypi

- hosts: master
  become: yes
  roles:
    - geoff_coppertop.k3s_roles.k3s.master

- hosts: node
  become: yes
  roles:
    - geoff_coppertop.k3s_roles.k3s.node

```

Note that instead of using the fully qualified collection name (FQCN) the following could be used instead,

```yml
- hosts: cluster
  collections:
  - geoff_coppertop.k3s_roles
  become: yes
  gather_facts: yes
  roles:
  - prereq
  - download
  - raspberrypi
  ```

Now that the roles are in an easily consummable format they can be easily integrated into other playbooks.

For the roles to function an inventory file with group(s) named as follows is necessary,

```ini
[master]
192.168.1.26
```

Other groups can be named as needed.

## Kubeconfig

To get access to your **Kubernetes** cluster just

```bash
scp debian@master_ip:~/.kube/config ~/.kube/config
```
