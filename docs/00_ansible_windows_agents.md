# This Lab will use Ansible to dowanload, deploay and Configure PPDM Agents using ansible

## Create and inventory File
ssh into the ubuntu JumpHost
create the inventory file as ~/workspace/hosts.yaml

```yaml
windows:
  children:
    databasehosts:
      hosts:
        file:
         ansible_host: file.demo.local
        sql01:
        sql02:
      vars:
        app_agent: "{{ msapp_agent_windows }}"
        app_agent_product_id: '{165F48F4-38C4-49DF-904E-4A23D39078BF}'
    filehosts:
      hosts:
        file:
      vars:
        app_agent: "{{ fs_agent_windows }}"
        app_agent_product_id: '{49C974B3-BA72-485C-9A96-794B065339DF}'
  vars:
    ansible_user: administrator #@demo
    ansible_password: Password123!
    ansible_winrm_scheme: https
    ansible_winrm_server_cert_validation: ignore
    ansible_shell_type: cmd
```

## Download Agents from PPDM Host to JumpBox
```bash
ansible-playbook ansible_ppdm/99_download_agents.yml -i hosts.yaml --extra-vars "ppdm_fqdn=ppdm-1.demo.local ppdm_new_password='Password123!'"
```

## Validate agent host ping-able vi winrm

```bash
ansible-playbook ansible_ppdm/99_download_agents.yml -i hosts.yaml --extra-vars "ppdm_fqdn=ppdm-1.demo.local ppdm_new_password='Password123!'"
```

## deploy agents to host "file"
```bash

ansible-playbook ansible_ppdm/99_download_agents.yml -i hosts.yaml --extra-vars "ppdm_fqdn=ppdm-1.demo.local ppdm_new_password='Password123!'"
```
## Approve Windows Agent

```bash
ansible-playbook ansible_ppdm/99_download_agents.yml -i hosts.yaml --extra-vars "ppdm_fqdn=ppdm-1.demo.local ppdm_new_password='Password123!'"
```
