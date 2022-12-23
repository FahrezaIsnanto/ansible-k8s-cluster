# Ansible - Kubernetes - On Premise - Setup

Kubernetes cluster pada Centos7 menggunakan Ansible,
Mesin yang dibutuhkan

1. Ansible controller - Laptop (Control Node)
2. Kubernetes Master - master.dc.com - 10.0.10.185 - 2 vcpu / 4 gib ram
3. Kubernetes Node1 - nodeone.dc.com - 10.0.10.186 - 1 vcpu / 4 gib ram
4. Kubernetes Node2 - nodetwo.dc.com - 10.0.10.187 - 1 vcpu / 4 gib ram
5. Kubernetes Node3 - nodethree.dc.com - 10.0.10.188 - 1 vcpu / 4 gib ram

## LANGKAH MENJALANKAN

### 1. Ubah file /etc/hosts pada ansible controller

```
10.0.10.185 master.dc.com
10.0.10.186 nodeone.dc.com
10.0.10.187 nodetwo.dc.com
10.0.10.188 nodethree.dc.com
```

### 2. Copy file host ansible controller ke semua k8snode

```
scp /etc/hosts root@10.0.10.185:/etc/
scp /etc/hosts root@10.0.10.186:/etc/
scp /etc/hosts root@10.0.10.187:/etc/
scp /etc/hosts root@10.0.10.188:/etc/
```

### 3. copy ssh public key ke semua authorized key k8snode

```
ssh-copy-id -i ~/.ssh/id_rsa.pub root@master.dc.com
ssh-copy-id -i ~/.ssh/id_rsa.pub root@nodeone.dc.com
ssh-copy-id -i ~/.ssh/id_rsa.pub root@nodetwo.dc.com
ssh-copy-id -i ~/.ssh/id_rsa.pub root@nodethree.dc.com
```

### 4. Ubah konfigurasi ansible

```
ubah file hosts
ubah ansible.cfg - inventory = /Users/fahreza/coding/ansible/centosk8s/code/hosts
```

### 5. Cek semua ansible inventory

```
ansible all --list-host
```

### 6. Jalankan ansible playbook (urut)

```
ansible-playbook k8s-pkg.yml
ansible-playbook k8s-master.yml
ansible-playbook k8s-workers.yml
```

### (Optional) Install Metallb

### Install nginx ingress controller dengan manifest

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml
```

### Install metallby dengan manifest

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
```

### buat manifest metallb-ingress.yaml

```
---

apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
name: default
namespace: metallb-system
spec:
addresses:

- 203.0.113.10-203.0.113.15
  autoAssign: true

---

apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
name: default
namespace: metallb-system
spec:
ipAddressPools:

- default
```

### Jalankan manifest

```
kubectl create -f metallb-ingress.yaml
```

### Test pastikkan external ip terisi

```
kubectl -n ingress-nginx get svc
```
