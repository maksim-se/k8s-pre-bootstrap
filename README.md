## Role info

> This playbook is not for fully setting up a Kubernetes Cluster.

It only helps you automate the standard Kubernetes bootstrapping pre-reqs.

## Supported OS

- CentOS 7
- CentOS 8 -- Still uses CentOS 7 repository ( will be updated when k8s repos for 8 is available)

## Tasks in the role

This role contains tasks to:

- Install basic packages required
- Setup standard system requirements - Disable Swap, Modify sysctl, Disable SELinux
- Install and configure a container runtime of your Choice - cri-o, Docker, Containerd
- Install the Kubernetes packages - kubelet, kubeadm and kubectl
- Configure Firewalld on Kubernetes Master and Worker nodes

## How to use this role

- Clone the Project:

```
$ git clone https://github.com/jmutai/k8s-pre-bootstrap.git
```

- Update your inventory, e.g:

```
$ vim hosts
[k8s-nodes]
172.21.200.10
172.21.200.11
172.21.200.12
```

- Update variables in playbook file

```
$ vim k8s-prep.yml
---
- name: Setup Proxy
  hosts: k8s-nodes
  remote_user: root
  #become: yes
  #become_method: sudo
  #gather_facts: no
  vars:
    k8s_cni: calico                                      # calico, flannel
    container_runtime: docker                            # docker, cri-o, containerd
    configure_firewalld: true                            # true / false
    # Docker registry
    setup_proxy: false                                   # Set to true to configure proxy
    proxy_server: "proxy.example.com:8080"               # Proxy server address and port
    docker_proxy_exclude: "localhost,127.0.0.1"          # Adresses to exclude from proxy
  roles:
    - kubernetes-bootstrap
```

If you are using non root remote user, then set username and enable sudo:

```
become: yes
become_method: sudo
```

To enable proxy, set the value of `setup_proxy` to `true` and provide proxy details.

## Running Playbook

Once all values are updated, you can then run the playbook against your nodes.

**NOTE**: For firewall configuration to open relevant ports for master and worker nodes, a pattern in hostname is required.

Check file:

```
$ vim roles/kubernetes-bootstrap/tasks/configure_firewalld.yml
....
- name: Configure firewalld on master nodes
  firewalld:
    port: "{{ item }}/tcp"
    permanent: yes
    state: enabled
  with_items: '{{ k8s_master_ports }}'
  when: "'master' in ansible_hostname"

- name: Configure firewalld on worker nodes
  firewalld:
    port: "{{ item }}/tcp"
    permanent: yes
    state: enabled
  with_items: '{{ k8s_worker_ports }}'
  when: "'node' in ansible_hostname"
```

If your master nodes doesn't contain `master` and nodes doesn't have `node` as part of hostname, update the file to reflect your naming pattern
