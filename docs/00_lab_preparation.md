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
password='Password123!'           
sudo adduser "user01" <<EOF
$password
$password
EOF
sudo su

cat <<EOF > /etc/sudoers.d/user01
user01 ALL=NOPASSWD: ALL
EOF

NEEDRESTART_SUSPEND=1 sudo apt-get update && sudo apt-get upgrade -y
NEEDRESTART_SUSPEND=1 sudo apt install -y python3-pip direnv zsh gnupg software-properties-common apt-transport-https ca-certificates gnupg curl sudo jq
sudo su user01
cd ~
python3 -m pip install --user ansible pywinrm pyyaml jinja2 kubernetes openshift jmespath
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
source ~/.zshrc
sed -i '/# export PATH=*/c\export PATH=$HOME/bin:/usr/local/bin:$HOME/.local/bin:$PATH' ~/.zshrc
mkdir workspace && cd "$_"
git clone https://github.com/bottkars/ansible_ppdm
git clone https://github.com/bottkars/ansible_ppdd
```
