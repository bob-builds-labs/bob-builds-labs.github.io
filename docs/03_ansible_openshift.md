# This Lab will use Ansible to configure and onboard an Openshift Cluster

Make sure openshift 3-Node Cluster Runs via starting from the Openshift Folder on vCenter:

![image](https://github.com/bob-builds-labs/bob-builds-labs.github.io/assets/8255007/16ada419-3e32-4324-912c-50ca260df14f)


## Onboard Openshift into PPDM

### Load the Environment
Change into lab 3 and allow the environment variables to load via direnv

```bash
cd ~/workspace/0499/lab3
direnv allow .
```

### create k8s policy and Rule

```bash
ansible-playbook ~/workspace/ansible_ppdm/130.0_playbook_add_k8s_policy_and_rule.yml
```
![image](https://github.com/bob-builds-labs/bob-builds-labs.github.io/assets/8255007/48dab02b-3a55-474f-a325-77a653fca353)


### Enable RBAC and onboard the cluster
```bash
ansible-playbook ~/workspace/ansible_ppdm/130.1_playbook_rbac_add_k8s_to_ppdm.yml -e '{"details": {"k8s": {"distributionType": "VANILLA_ON_VSPHERE","vCenterId": "aee5f921-d2c6-5d5d-bfe7-e031e8241d2b"}}}'
```
![image](https://github.com/bob-builds-labs/bob-builds-labs.github.io/assets/8255007/a787dff5-1803-4733-8a59-2217a4b912c9)
