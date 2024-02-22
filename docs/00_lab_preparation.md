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
NEEDRESTART_SUSPEND=1 sudo apt install -y python3-pip direnv zsh gnupg software-properties-common apt-transport-https ca-certificates gnupg curl sudo jq tree
### as root
wget https://vcenter01.demo.local/certs/download.zip --no-check-certificate
unzip download.zip
mkdir -p /usr/share/ca-certificates/extra
cp certs/lin/*.0  /usr/share/ca-certificates/extra
cd /usr/share/ca-certificates/extra
for file in *.0; do mv "$file" "$(basename "$file" .0).crt"; done
update-ca-certificates
sudo dpkg-reconfigure ca-certificates
# GUI will come up#  Select 'Yes' to trust new certs#  Select checkbox for certs added to 'extra' directory
# verify cert from 'extra' was added

### as admin user
python3 -m pip install --user ansible pywinrm pyyaml jinja2 kubernetes openshift jmespath
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
sed -i '/# export PATH=*/c\export PATH=$HOME/bin:/usr/local/bin:$HOME/.local/bin:$PATH' ~/.zshrc
direnv hook zsh >> ~/.zshrc
source ~/.zshrc

/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install openshift-cli  kubectl k9s govc

mkdir workspace && cd "$_"
git clone https://github.com/bottkars/ansible_ppdm
git clone https://github.com/bottkars/ansible_ppdd

ssh-keygen -t ed25519 -N '' -f ~/.ssh/openshift
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
tar xzfv openshift-install-linux.tar.gz
```
