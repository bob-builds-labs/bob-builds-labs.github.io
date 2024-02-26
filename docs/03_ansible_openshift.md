# This Lab will use Ansible to configure and onboard an Openshift Cluster



if not already done, clone into https://github.com/bob-builds-labs/0499.git
```bash
git clone https://github.com/bob-builds-labs/0499.git
```
### Load the Environment
Change into lab 3 and allow the environment variables to load via direnv

```bash
cd ~/workspace/0499/lab3
direnv allow .
```

Start the OpenShift 3-Node Cluster  from the Openshift Folder on vCenter or via govc: 

```bash
govc find / -type m -name "openshift*master-*" | xargs -I % govc vm.power -on -vm.ipath=%
```

![image](https://github.com/bob-builds-labs/bob-builds-labs.github.io/assets/8255007/1f3f196e-9780-4989-a2b4-90da7a4361c2)

It will take some time for the Nodes to Start.

Validate the nodes are Ready:
```bash
oc get nodes -l node-role.kubernetes.io/master
```

If the nod3es are NotReady, there might be autstanding Certificate Signing requests.  
Check with  
```bash
oc get csr
```
To approve pending Requests, run

```bash
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}'
```


Wait for https://console-openshift-console.apps.openshift.demo.local to be reachable
It might take a while for the cluster to reconcile
## Onboard Openshift into PPDM



### create k8s policy and Rule

```bash
ansible-playbook ~/workspace/ansible_ppdm/130.0_playbook_add_k8s_policy_and_rule.yml
```
![image](https://github.com/bob-builds-labs/bob-builds-labs.github.io/assets/8255007/0a94f626-1c4d-4b6a-be03-64608f6347ec)


### Enable RBAC and onboard the cluster
```bash
ansible-playbook ~/workspace/ansible_ppdm/130.1_playbook_rbac_add_k8s_to_ppdm.yml -e '{"details": {"k8s": {"distributionType": "VANILLA_ON_VSPHERE","vCenterId": "aee5f921-d2c6-5d5d-bfe7-e031e8241d2b"}}}'
```
![image](https://github.com/bob-builds-labs/bob-builds-labs.github.io/assets/8255007/0c8da6a2-48d1-4320-ad04-65f9a3aa6fba)



