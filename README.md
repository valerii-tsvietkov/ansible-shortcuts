# ansible-shortcuts
Useful tips, tricks, code snippets when working with Ansible

This repository contain ready to start Vagrant and Ansible settings which help You provision Ansible development lab on Your regular Windows PC.  
See [step by step guide](documentation/development/) how You can use that. Beside this I plan to copy here some of my thought and findings according Ansible topic.

## Content
* [Local Dev/Sandboxing Environment and development principles](documentation/development/)
* [Ansible Introduction](documentation/intro/)
* [Ubuntu without python](snippets/ubuntu_no_python.yml)
* [custom Bind for Dev environments](roles/bind/) enhanced version of role which create DNS zone files from Ansible inventory 
* [Triks](documentation/triks/). List of "A-HA" moments.
## Installation of test project

After finishing [step by step guide](documentation/development/) run `vagrant up` in GitBash or Powershell when locating in root folder of this repo.   

Read Logs on screen

You should mention:  
 - VM creation
 - acontroller will recieve ansible installation
 - github Ansible roles download from external repositories
 - Ansible will install Bind Apache and one page website on one VM `server1`
 - Ansible will test installation from VM `client1`
 
You can visit [http://192.168.56.11/](http://192.168.56.11/) To see this test page served from Apache on Your VM.  
This page is not aviable from outside world.  

Now You can modify and upgrade this test project according to Your case
