# This Lab will use Ansible to configure and onboard an Openshift Cluster

Start the OpenShift 3-Node Cluster  from the Openshift Folder on vCenter:
![image](https://github.com/bob-builds-labs/bob-builds-labs.github.io/assets/8255007/1f3f196e-9780-4989-a2b4-90da7a4361c2)

Wait for https://console-openshift-console.apps.openshift.demo.local to be reachable
It might take a while for the cluster to reconcile

if not already done, clone into https://github.com/bob-builds-labs/0499.git
```bash
git clone https://github.com/bob-builds-labs/0499.git
```

## Onboard Openshift into PPDM

### Load the Environment
Change into lab 3 and allow the environment variables to load via direnv

```bash
cd ~/workspace/0499/lab3
direnv allow .
```

### create k8s policy and Rule

```bash
$HOME/workspace/ansible-playbook ~/workspace/ansible_ppdm/130.0_playbook_add_k8s_policy_and_rule.yml
```
![image](https://github.com/bob-builds-labs/bob-builds-labs.github.io/assets/8255007/48dab02b-3a55-474f-a325-77a653fca353)


### Enable RBAC and onboard the cluster
```bash
$HOME/workspace/ansible-playbook ~/workspace/ansible_ppdm/130.1_playbook_rbac_add_k8s_to_ppdm.yml -e '{"details": {"k8s": {"distributionType": "VANILLA_ON_VSPHERE","vCenterId": "aee5f921-d2c6-5d5d-bfe7-e031e8241d2b"}}}'
```
![image](https://github.com/bob-builds-labs/bob-builds-labs.github.io/assets/8255007/a787dff5-1803-4733-8a59-2217a4b912c9)
