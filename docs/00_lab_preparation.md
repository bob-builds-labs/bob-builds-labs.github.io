# Lab 0499 Automation Preparation

## Preparation of Windows Hosts

```Powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
choco install python --version=3.9.0 --confirm
iex ((New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/ansible/ansible/stable-2.12/examples/scripts/ConfigureRemotingForAnsible.ps1'))
restart-computer 
```

## Preparation of Launchpad Host


## Ubuntu Box

```bash
#!/bin/bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install -y python3-pip direnv zsh
python3 -m pip install --user ansible pywinrm
python3 -m pip install --user pyyaml kubernetes openshift
sudo sudo apt-get install -y gnupg software-properties-common apt-transport-https ca-certificates gnupg curl sudo
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
direnv hook zsh >> 
source ~/.zshrc
mkdir workspace && cd "$_"
git clone https://github.com/bottkars/ansible_ppdm
git clone https://github.com/bottkars/ansible_ppdd
```
