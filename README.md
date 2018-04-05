# ansible-shortcuts
Useful tips, tricks, code snippets when working with Ansible

## Content
* [Local Dev/Sandboxing Environment and development principles](documentation/development/)
* [Ubuntu without python](snippets/ubuntu_no_python.yml)
* [custom Bind for Dev environments](roles/bind/) enhanced version of role which create DNS zone files from Ansible inventory 

### OpenStack + Ansible
Beside Provisioner it can be (Terraform, Heat, Ansible or just manual creation of VM instance) we use so called
[Dynamic Inventory](http://docs.ansible.com/ansible/intro_dynamic_inventory.html#example-openstack-external-inventory-script)
this inventory script relay on instance metadata "group" or "groups" property. For example one instance can be with "group"="dnsserv"
