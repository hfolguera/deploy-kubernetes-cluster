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
3 or more machines must be accessible to deploy the kubernetes environment, so I use my local VMWare ESXi environment to provision the required virtual machines.

There are a lot posts talking discussing to deploy a k8s cluster on a virtual environment vs bare metal. In my case, I have decided to go for a virtual environment, since it is a laboratory and I can quickly destroy the VM and recreate it. Also, I can shutdown some nodes when I need more resources.

In my ESXi lab, I create 3 copies of my Oracle Linux 7 template (OL7.7) and configure the following:
1. Hostname, IP Address and DNS (through Network Manager `nmtui`).

### Kubernetes cluster deployment
Once all the requirements are met and the final virtual machines have the minimum configuration, it's time to execute the Ansible playbook to deploy the cluster.
First of all, you need to modify the inventory `hosts` to point out your machine names or IP addresses. Follow the given example.

Once the inventory is configured, you need to execute the playbook with the following command:
`ansible-playbook -i hosts deploy-k8s-cluster.yml`

*Important!* Remember this is a idempotent playbook! That means that if you find any issue, you can fix it and execute again the playbook from the beggining.

### Untaint master node
By default, master node does not schedule pods for security and QoS reasons. This behaviour can be altered with the following command:
```
kubectl taint nodes <master_node_name> node-role.kubernetes.io/master-
```

## Next steps

## Feedback and Contributing
If you find any issue on this playbook, please open an issue or a PR and I'll check it. Thanks!
