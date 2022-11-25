# Setting up a Kubernetes cluster with Vagrant and Ansible provisioning

The build works only with Vmware Desktop/Fusion Virtualization Tool on Linux/Mac OS. Windows OS has a limitation related to Ansible provisioning usage.

Vagrant as default assigns NAT interface to guest VM and the interface can be reached from host if Vmware Desktop/Fustion tool is used. Virtualbox is designed to not expose NAT interface to the host. 

In addition to `vagrant provision` method the ansible can be executed from host using following command:

```console
user:~/kubernetes-vagrant-cluster$ ansible-playbook build/playbook.yml -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory
```

The build as default starts one master node and one slave node. To start more slave nodes the following variable need to be modified in `Vagrantfile`:
```C
node = 1
```

> **Warning**
>  Only one master node is supported in the build. Please do not increase `master` variable

Each cluster nodes is configured to have 2 CPU and 4096 RAM memory. The configuration can be changed in `Vagrantfile` in section:

```C
machine.vm.provider :vmware_fusion do |vmware|
  vmware.vmx["numvcpus"] = 2
  vmware.vmx["memsize"] = 4096
end
```

To start using K8S just log in into guest VM:

```console
user:~/kubernetes-vagrant-cluster$ vagrant ssh k8s_master_1

vagrant@k8s-master-1:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
k8s-master-1   Ready    control-plane   3d    v1.25.4
k8s-node-1     Ready    <none>          3d    v1.25.4
```

The K8S cluster can be also reached from host using K8s configuration file:

```console
user:~/kubernetes-vagrant-cluster$ kubectl --kubeconfig k8s.config <options>

# E.g
user:~/kubernetes-vagrant-cluster$ kubectl --kubeconfig k8s.config get nodes
NAME           STATUS   ROLES           AGE   VERSION
k8s-master-1   Ready    control-plane   3d    v1.25.4
k8s-node-1     Ready    <none>          3d    v1.25.4

user:~/kubernetes-vagrant-cluster$ kubectl --kubeconfig k8s.config get pods --all-namespaces
NAMESPACE      NAME                                   READY   STATUS    RESTARTS      AGE
kube-flannel   kube-flannel-ds-tpw4f                  1/1     Running   8 (18m ago)   3d
kube-flannel   kube-flannel-ds-wf8p9                  1/1     Running   6 (19m ago)   3d
kube-system    coredns-565d847f94-bhlcv               1/1     Running   6 (19m ago)   3d
kube-system    coredns-565d847f94-h6sjh               1/1     Running   6 (19m ago)   3d
kube-system    etcd-k8s-master-1                      1/1     Running   6 (19m ago)   3d
kube-system    kube-apiserver-k8s-master-1            1/1     Running   6 (19m ago)   3d
kube-system    kube-controller-manager-k8s-master-1   1/1     Running   6 (19m ago)   3d
kube-system    kube-proxy-575sf                       1/1     Running   6 (18m ago)   3d
kube-system    kube-proxy-tbd7g                       1/1     Running   6 (19m ago)   3d
kube-system    kube-scheduler-k8s-master-1            1/1     Running   6 (19m ago)   3d
```

There is addiotnal tools installed on node master guest VM:
- **`Helm`** - the package manager for Kubernetes (https://helm.sh)
- **`k9s`** -  the terminal based UI to interact with Kubernetes clusters (https://k9scli.io)

The tools can be used from the host as well:
* **`Helm`**
```console
user:~/kubernetes-vagrant-cluster$ helm --kubeconfig k8s.config <parameters>

# E.g
user:~/kubernetes-vagrant-cluster$ helm --kubeconfig k8s.config install bitnami/nginx --generate-name
```

* **`k9s`**
```console
user:~/kubernetes-vagrant-cluster$ k9s --kubeconfig k8s.config
```

To configure **`Lens - The Kubernetes IDE`** (https://k8slens.dev) to connect to the cluster, import k8s.config file.
