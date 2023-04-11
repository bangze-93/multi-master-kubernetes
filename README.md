## Deploy multi master Kubernetes cluster using Ansible on Ubuntu 20
This Kubernetes cluster use Keepalived for High Availability (HA)
##### Kubernetes master
- master1 : 192.168.200.21
- master2 : 192.168.200.22
- master3 : 192.168.200.23
##### Kubernetes worker
- worker1 : 192.168.200.24
- worker2 : 192.168.200.25
- worker3 : 192.168.200.26
- worker4 : 192.168.200.27
- worker5 : 192.168.200.28
##### Keepalived
- vip : 192.168.200.30
- lb1 : 192.168.200 31
- lb2 : 192.168.200.32

To deploy cluster, first configure keepalived and than configure kubernetes
### Configure Keepalived
Adjust to your env (playbook.yml in vars section)
```bash
  vars:
    - ip_virtual: 192.168.200.30
    - ip_lb1: 192.168.200.31
    - ip_lb2: 192.168.200.32
    - interface: "{{ ansible_default_ipv4.interface }}"
    - ip_master1: 192.168.200.21
    - ip_master2: 192.168.200.22
    - ip_master3: 192.168.200.23
```
Run playbook

Change user [deploy] to your server user
```bash
ansible-playbook -i hosts playbook.yml -u deploy
```


### Configure Kubernetes
Make sure to use same IP in playbook.yml and join-cluster.yml (endpoint_ip) as in playbook.yml keepalived (ip_virtual) 192.168.200.30
##### playbook.yml
```bash
  vars:
    - kube_version: 1.25.8-00
    - endpoint_ip: 192.168.200.30
    - network_cidr: 10.3.4.0/16
```
##### join-cluster.yml
```bash
  vars:
    - endpoint_ip: 192.168.200.30
```
Run playbook

Change user [deploy] to your server user
```bash
ansible-playbook -i hosts playbook.yml -u deploy
```
Jot down the output at task "Get join key"
```bash
You can now join any number of the control-plane node running the following command on each as root:
   kubeadm join 192.168.200.30:6443 --token 12ytci.c0elqb4mxpowvnpn \
      --discovery-token-ca-cert-hash sha256:6d77b7a8c233d134cb0e027d0dbc8bd7956fca660d486e5dcd92ce6b33f09928 \
      --control-plane --certificate-key 59c605a446bf25923031153baa6463e1a05559aa857bce7baac150c87b62c515

Then you can join any number of worker nodes by running the following on each as root:
   kubeadm join 192.168.200.30:6443 --token 12ytci.c0elqb4mxpowvnpn \
      --discovery-token-ca-cert-hash sha256:6d77b7a8c233d134cb0e027d0dbc8bd7956fca660d486e5dcd92ce6b33f09928
```
Change join key in join-cluster.yml with that output
```bash
    - name: Join for master
      ansible.builtin.shell: |
        kubeadm join {{ endpoint_ip }}:6443 --token 12ytci.c0elqb4mxpowvnpn \
            --discovery-token-ca-cert-hash sha256:6d77b7a8c233d134cb0e027d0dbc8bd7956fca660d486e5dcd92ce6b33f09928 \
            --control-plane --certificate-key 59c605a446bf25923031153baa6463e1a05559aa857bce7baac150c87b62c515
    ....
    ....
    - name: Join for worker
      ansible.builtin.shell: |
        kubeadm join {{ endpoint_ip }}:6443 --token 12ytci.c0elqb4mxpowvnpn \
            --discovery-token-ca-cert-hash sha256:6d77b7a8c233d134cb0e027d0dbc8bd7956fca660d486e5dcd92ce6b33f09928
```
Run playbook to join cluster
```
ansible-playbook -i hosts join-cluster.yml -u deploy
```
