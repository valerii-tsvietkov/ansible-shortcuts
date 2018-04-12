## Content
* Local Dev/Sandboxing Environment
* Git
* Principle of local sandboxing
* Configuration
* Contributing Guide
* Acknowledgments

## Local Dev/Sandboxing Environment

There is a `Vagrantfile` provided in the repository root to allow for easy provisioning of a local (on your PC) dev environment to experiment.
### Prerequisites (please install in mention order)
* [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 
* [Oracle Virtualbox](https://www.virtualbox.org/wiki/Downloads)
* [Notepad++](https://notepad-plus-plus.org/download)
* [Git](https://git-scm.com/downloads).  
    #### Custom install options:  
    * screen2: Select Git bash Here;
    * screen3: (Optional) You can select Notepad++ as commit messages editor
    * screen4: Use git from Git bash only
    * screen5: Use tortouse plink - allow integration with putty agent and SSH keys. Check plink path in Putty installation
    * screen6: OpenSSL
    * screen7: UnixStyle Line ending
    * screen8: Default MinTTY
    * Next, Next, Install
* [Vagrant](https://www.vagrantup.com/downloads.html) 

### Generate SSH key for GIT repo
You can use Git with Putty Pageant and not give passwords to Git programm when interacting with repo.  
Generate dedicated key in Puttygen. Save private part with password.  
Load public part to Your account
Use privatekey with pageant. 
Run Putty and try to connect to git@<repo_ssh_url>. Save repository signature.  
Connection will fail it is OK. Purpose of this connection is to save SSH key of repo to known keys- git bash have bug which not allow doing this.  
Run [pageant](https://www.ssh.com/ssh/putty/putty-manuals/0.68/Chapter9.html) with right key before interacting with repo via GitBash or SSH from putty to controller VM.

## Git
Configure Git for the first time from gitbash
```
git config core.autocrlf input # To be sure that Unix line ending will applied
git config user.name "Your, Name" # Change name
git config user.email "your@email" # Change email 
```
 
## Principle of local sandboxing
#### Vagrant provision(create) VMs

```
# In repository directory
$ vagrant status
Handling vm with hostname [server1] and IP [192.168.56.11]
Handling vm with hostname [client1] and IP [192.168.56.12]
Current machine states:

server1                   not created (virtualbox)
client1                   not created (virtualbox)
acontroller               not created (virtualbox)

# Create all environment and launch
$ vagrant up
```

#### Suspend and resume after initial up
To halt the all running VMs before shutdown Your PC or when You are not developing:  
`vagrant suspend`  

Come back to development next day  
`vagrant resume`  

To stop all VMs and purge it from Oracle Vbox:  
`vagrant destroy -f`  

Problem with disck space? You can delete some unused Vagrant boxes with  
`vagrant box`   

You can also halt (shutdown) machines, see the status of all machines, validate the configuration and many things more:  
`vagrant help`

#### Putty session settings
`Host:` 192.168.56.2  
`Session name:` controller  
`Connection/SSH/Auth/`: (Add checkmark to `Authentification parameters:` [x] Allow agent forwarding)  
Save session settings.  
This will give You ability to interact with other repos from Ansible playbooks which running inside Vagrant controller VM. 

#### Vagrant provision(install software) on VMs
Provision steps for VM `controller` include istallation of developer tools and settings. Look inside [Vagrantfile]Vagrantfile for explanations.  
SSH to controller VM using Putty (192.168.56.2 L/P vagrant/vagrant)  
Check "Forward SSH agent" Putty option if You want be able to do git commit with ssh key from inside VM.  

```
cd /vagrant # Root directory of the repo
```
use `screen` `vim` and `git` for developing and saving from inside controller VM. If You like Linux like myself.
Ansible playbook command for testing Your changes:   
```
ansible-playbook site.yml # -l "node1," Limit selection if You spin up only part of the VMs
```

## **Configuration**
Quantity and IPs of Virtualbox VMs controlled by [Vagrantfile](../../Vagrantfile).
`Vagrantfile`:
This is the base Vagrant configuration plus some enhanced logic for our multi machine setup. 
All changes to that file should be reflected to [inventory/vagrant/hosts](../../inventory/vagrant/hosts)

## Contributing Guide

Main principles writing/editing roles
 * Idempodency 
(each subsequent run should not change already changed state. 
Check desired result before modification)

 * All environment specifyc host data should come from inventory system. 
Or be defined dynamicaly via runtime.
wisely use "__group_vars/__" and do not use __host_vars/__ at all because 
__inventory_hostname__ can be changed with time.

## Acknowledgments
* Thanks [docs.ansible.com](http://docs.ansible.com) for outstanding explanation best practices and examples.
* Thanks [Vagrant](https://www.vagrantup.com/docs/index.html) for useful code snippets
* Thanks to developers who created re-usable roles.
* Thanks curious people who describe problems and solutions to community
