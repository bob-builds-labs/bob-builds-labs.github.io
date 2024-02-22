# This Lab will use Ansible to download, deploy and configure PPDM Agents and Registrations

## The Inventory File

ansible communicates with remote hosts using ssh / winrm. Remote hsost do not require ansible to be deployed, but Python.
Python is already configured on the Remote Hosts. 

Remote Hosts are treated as "Inventory", that ansible will Gather "Facts" from ( OS releated information, Network Softaware etc )

We can define our Inventory via Invetory Files. Most Likely. Inventory files are stored in a GitOps Structure

### Create the below inventory File by Copying the Code 

```bash
cat <<EOF > hosts.yaml
windows:    # <- a group of Hosts addressed as windows
  children:
    databasehosts:  # <- a group of Hosts addressed as databasehosts
      hosts:
        sql01: # <- an indivisual Host adressed as sql01
        sql02:
          ansible_host: sql02.demo.local # <- an indivisual Host adressed as sql02. resolvable as sql02.demo.local
      vars: # specific Vars for the Group Database Hosts
        app_agent: "{{ msapp_agent_windows }}"
        app_agent_product_id: '{165F48F4-38C4-49DF-904E-4A23D39078BF}'
    filehosts:
      hosts:
        file:
      vars:
        app_agent: "{{ fs_agent_windows }}"
        app_agent_product_id: '{49C974B3-BA72-485C-9A96-794B065339DF}'
    exchangehosts:
      hosts:
        exchange1:
        exchange2:
      vars:
        app_agent: "{{ exchange_agent_windows }}"
        app_agent_product_id: '{165F48F4-38C4-49DF-904E-4A23D39078BF}'
  vars: # -> vars for all Windows Hosts
    ansible_user: administrator
    ansible_password: Password123!
    ansible_winrm_scheme: https
    ansible_winrm_server_cert_validation: ignore
    ansible_shell_type: cmd
    ansible_connection: winrm
EOF

```


### Validate agent host "file"  ping-able vi winrm

```bash
ansible file -m win_ping -i hosts.yaml
```



### Download Agents from PPDM Host to JumpBox
```bash
ansible-playbook ansible_ppdm/100.0_download_agents.yml --extra-vars "ppdm_fqdn=ppdm-1.demo.local ppdm_new_password='Password123!'"
```


### configure PPDM to use DNS for name Resolution
as we will register the agents to ppdm using the fqdn, we need to enable DNS for Agents on the PPDM Host:

```bash
ansible-playbook ansible_ppdm/100.2_set_dns_for_agents.yaml --extra-vars "ppdm_fqdn=ppdm-1.demo.local ppdm_new_password='Password123!'"
```

## Installing the File Agent

### deploy agents to host "file"
The example will use the Inventory from the previously create file "hosts.yaml"
As we honly want to tackle the host named file, we use the --limit parameter

```bash
ansible-playbook ansible_ppdm/100.1-playbook-copy-and-deploy-windows-agent.yaml -i hosts.yaml --limit file, -e ppdm_fqdn=ppdm-1.demo.local
```

### View Agent registration Status

```bash
ansible-playbook ansible_ppdm/100.5_get_agent_registration_status.yaml --extra-vars "ppdm_fqdn=ppdm-1.demo.local ppdm_new_password='Password123!'"
```


### Approve File Agent

```bash
ansible-playbook ansible_ppdm/100.3_create_whitelistentry.yaml --extra-vars "ppdm_fqdn=ppdm-1.demo.local ppdm_new_password='Password123!'" -e '{ "host_list" : [ "file.demo.local" ] }'
```

