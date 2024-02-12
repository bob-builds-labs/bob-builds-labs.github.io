# This Lab will use Ansible to download, deploy and configure PPDM Agents using ansible

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
    ansible_user: administrator
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
## configure PPDM to use DNS for name Resolution
as we will register the agents to ppdm using the fqdn, we need to enable DNS for Agents on the PPDM Host:

```bash
ansible-playbook ansible_ppdm/100.1-playbook-copy-and-deploy-windows-agent.yaml -i hosts.yaml --limit file, -e ppdm_fqdn=ppdm-1.demo.local
```

## deploy agents to host "file"
The example will use the Inventory from the previously create file "hosts.yaml"
As we honly want to tackle the host named file, we use the --limit parameter
```bash
ansible-playbook ansible_ppdm/100.1-playbook-copy-and-deploy-windows-agent.yaml -i hosts.yaml --limit file, -e ppdm_fqdn=ppdm-1.demo.local
```
## Approve Windows Agent

```bash
ansible-playbook ansible_ppdm/99_download_agents.yml -i hosts.yaml --extra-vars "ppdm_fqdn=ppdm-1.demo.local ppdm_new_password='Password123!'"
```
