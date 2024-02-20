# This Lab will use Ansible to configure and onboard an Openshift Cluster

Make Sure Opensift 3-Node Cluster Runs via starting from the Openshift Folder on vSphere

## Onboard Openshit into PPDM

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


### Enable RBAC and obboard them cluster
```bash
ansible-playbook ~/workspace/ansible_ppdm/130.1_playbook_rbac_add_k8s_to_ppdm.yml
```
