# LEHACK 2024 (Active Directory pwnage with NetExec)

<div>
<img src="./lehack-2024.png"/>
</div>

Welcome to the NetExec Active Directory Lab! This lab is designed to teach you how to exploit Active Directory (AD) environments using the powerful tool NetExec.

Originally featured in the LeHack2024 Workshop, this lab is now available for free to everyone! In this lab, you’ll explore how to use the powerful tool NetExec to efficiently compromise an Active Directory domain during an internal pentest.

The ultimate goal? Become Domain Administrator by following various attack paths, using nothing but NetExec! and Maybe BloodHound (Why not :P)

> Obviously do not cheat by looking at the passwords and flags in the recipe files, the lab must start without user to full compromise

## Prerequisites

- Providing
  - [Vmware workstation](https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html)
  - [Vagrant](https://developer.hashicorp.com/vagrant/docs)
  - [Vmware utility driver](https://developer.hashicorp.com/vagrant/downloads/vmware)
  - Vagrant plugins:
    - vagrant-reload
    - vagrant-vmware-desktop
    - on some distribution also the vagrant plugins :
      - winrm
      - winrm-fs
      - winrm-elevated

- Provisioning with python
  - Python3 (>=3.8)
  - [ansible-core==2.12.6](https://docs.ansible.com/ansible/latest/index.html)
  - pywinrm

- Or provisioning With Docker
  - [Docker](https://www.docker.com/)


## Install dependencies

> No automatic install is provided as it depend of your package manager and distribution. Here are some install command lines are given for ubuntu.

### Install Vmware workstation

- Download and install vmware workstation and set your license key (or use the 30 days trial) [workstation-pro-evaluation](https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html)
- __Note that workstation-player can't manage clone and snapshot and will not work with vagrant you need to use the pro version__
- You will also need to install VMware Utility driver (https://developer.hashicorp.com/vagrant/downloads/vmware)
(an install guide can be found here : https://developer.hashicorp.com/vagrant/docs/providers/vmware/vagrant-vmware-utility)

```bash
cd /tmp
wget https://releases.hashicorp.com/vagrant-vmware-utility/1.0.21/vagrant-vmware-utility_1.0.21_linux_amd64.zip
sudo mkdir -p /opt/vagrant-vmware-desktop/bin
sudo unzip -d /opt/vagrant-vmware-desktop/bin vagrant-vmware-utility_1.0.21_linux_amd64.zip
sudo /opt/vagrant-vmware-desktop/bin/vagrant-vmware-utility certificate generate
sudo /opt/vagrant-vmware-desktop/bin/vagrant-vmware-utility service install
```

- Note that you will need to install the vmware-desktop plugin after the vagrant installation : 

```bash
vagrant plugin install vagrant-vmware-desktop
```

### Install Vagrant

- **vagrant** from their official site [vagrant](https://developer.hashicorp.com/vagrant/downloads). __The version you can install through your favorite package manager (apt, yum, ...) is probably not the latest one__.
- Install vagrant plugin vbguest if you want the guest addition: `vagrant plugin install vagrant-vbguest` (not mandatory)
- Vagrant installation is well describe in [the official vagrant page](https://developer.hashicorp.com/vagrant/downloads) (tests are ok on 2.3.4)
- Some github issues indicate vagrant got some issue on some version and work well with 2.2.19 (`apt install vagrant=2.2.19`)

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
```

- on some recent versions (ubuntu 23.04), you should consider run also:
```bash
gem install winrm winrm-fs winrm-elevated
```

### Install Ansible

- If you want to play ansible from your host or a linux vm you should launch the following commands :

- *Create a python >= 3.8 virtualenv*

```bash
sudo apt install git
git https://github.com/Pennyw0rth/NetExec-Lab.git
cd LEHACK/ansible
sudo apt install python3.8-venv
python3.8 -m virtualenv .venv
source .venv/bin/activate
```

- Install ansible and pywinrm in the .venv
  - **ansible** following the extensive guide on their website [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).
  - **Tested with ansible-core (2.12)**
  - **pywinrm** be sure you got the pywinrm package installed

```
python3 -m pip install --upgrade pip
python3 -m pip install ansible-core==2.12.6
python3 -m pip install pywinrm
```

- Install all the ansible-galaxy requirements
  - **ansible windows**
  - **ansible community.windows**
  - **ansible chocolatey** (not needed anymore)
  - **ansible community.general**
```
ansible-galaxy install -r ansible/requirements.yml
```

## Install

### Create the vms

- To create the VMs just run 

```bash
cd ad/LEHACK/providers/vmware
vagrant up
```

*note: For some distributions, you may need to run additional commands to install WinRM gems* this can be done via the following commands:

```bash
vagrant plugin install winrm
vagrant plugin install winrm-fs
vagrant plugin install winrm-elevated
```

- At the end of the vagrantup you should have the vms created and running
### Launch provisioning with Ansible

- launch the provision script (launch ansible with failover on errors)

```bash
cd ansible
export ANSIBLE_COMMAND="ansible-playbook -i ../ad/LEHACK/data/inventory -i ../ad/LEHACK/providers/vmware/inventory"
export LAB="LEHACK"
../scripts/provisionning.sh
```

- or launch ansible directly

```bash
cd ansible/
ansible-playbook -i ../ad/LEHACK/data/inventory -i ../ad/LEACK/providers/vmware/inventory main.yml
```
