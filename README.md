# deploy-kubernetes-cluster
Ansible playbook to deploy kubernetes cluster

## Cluster Deployment
Follow this steps to deploy a k8s cluster on a local environment.

### Requirements
Before executing the ansible playbook to deploy the cluster, you need to satisfy the following requirements:
1. Having a machine with ansible installed. I'm using my laptop since I already have ansible installed in it, but you can use any other vm.
2. The python's netaddr plugin must be installed in that machine since the playbook use the ipaddr filter. You can install it with `pip3 install netaddr` or remove it from the "Init Kubernetes Cluster" task and replacing it with /24 (or whatever netmask you need).
3. Register DNS entries for your cluster nodes or assign an IP address. I'll use DNS name resolution.
4. You must have 3 VMs or physical machines (or more) where you will deploy the k8s cluster and you must have passwordless access to those machines. You can simply copy your public ssh key to `/root/.ssh/authorized_keys`. In the next section I explain how I have deployed my VMs.

### Considerations
1. The Ansible playbook has been written to be deployed on machines based on Red Hat distributions.
2. Calico is the network plugin deployed. Please, consider modifying the code if you need a different one.

### VM deployment
3 or more machines must be accessible to deploy the kubernetes environment, so I use my local ~~VMWare ESXi~~ Proxmox environment to provision the required virtual machines.

There are a lot posts talking discussing to deploy a k8s cluster on a virtual environment vs bare metal. In my case, I have decided to go for a virtual environment, since it is a laboratory and I can quickly destroy the VM and recreate it. Also, I can shutdown some nodes when I need more resources.

In my ~~ESXi~~ Proxmox lab, I create 3 copies of my Oracle Linux 7 template (OL7.7) and configure the following:
1. Hostname, IP Address and DNS (through Network Manager `nmtui`).

### Ansible installation
I'm currently using a Macbook pro with M1 architecture...Ansible client can be installed with the following command:

```
brew update
brew install ansible
```

### Kubernetes cluster deployment
Once all the requirements are met and the final virtual machines have the minimum configuration, it's time to execute the Ansible playbook to deploy the cluster.
First of all, you need to modify the inventory `hosts` to point out your machine names or IP addresses. Follow the given example.

Once the inventory is configured, you need to execute the playbook with the following command:
`ansible-playbook -i hosts deploy-k8s-cluster.yml`

*Important!* Remember this is a idempotent playbook! That means that if you find any issue, you can fix it and execute again the playbook from the beggining.

## Post Deployment tasks

### Install Helm
Some applications are deployed using Helm. Install Helm with the following commands:
```
mkdir helm
cd helm
wget https://get.helm.sh/helm-v3.17.4-linux-amd64.tar.gz
tar -xzvf tar -xzvf helm-v3.17.4-linux-amd64.tar.gz
echo "export PATH=$PATH:helm/linux-amd64" >> .bash_profile
```
You can check the last version of Helm [here](https://github.com/helm/helm/releases)



### Configure storage provider
A default storage class will be defined in order to dynamically create PersistentVolumes. I'll take profit of a Synology to provide a NFS storage class.
As a requirement, Helm needs to be installed and configured.

```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=192.168.1.11 --set nfs.path=/volume2/k8s
kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

Verify the new storage class has been successfully configured with `kubectl get storageclass`.

### Configure a Load Balancer
I'll use the Metallb project as my custom Load Balancer.
Follow the instructions in the following repository: [metallb](https://github.com/hfolguera/metallb)

### Untaint master node
By default, master node does not schedule pods for security and QoS reasons. This behaviour can be altered with the following command:
```
kubectl taint nodes <master_node_name> node-role.kubernetes.io/control-plane:NoSchedule-
```

## Feedback and Contributing
If you find any issue on this playbook, please open an issue or a PR and I'll check it. Thanks!
