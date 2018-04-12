# Introduction to Ansible

> Disclaimer: This is advanced material for Live Demo or for those who already read [basic documentation](http://docs.ansible.com)

## Content:
- [Benefits of Ansible](#benefits-of-ansible)
- [Ansible Inventory](#ansible-inventory)
- [Variables](#variables)
- [Using Ansible with cloud solutions.](#using-ansible-with-cloud-solutions.)
- [Continuous Integration and Continuous Deployment with Ansible](#continuous-integration-and-continuous-deployment-with-ansible)

## Benefits of Ansible:
* Minimalistic easy to read YAML syntax
* Possibility to mix declarative and imperative style
* Clean obvious architecture
* Push model
* Python inside
* Client less. Only SSH/WinRM and python/powershell on other end is required

### Push model

My opinion about Puppet Pull configuration Orchestration model that it best suits for independent servers at scale. They connect to Master once in 30 min receive security updates and go sleep again.  
Maybe I am wrong and in Puppet also possible put some dependency and ordering who will receive what commands and when.   
But with Ansible push model it looks more straightforward how to control tight related on each other services deployment  

For example:  
*site.yml*
```
---
- import_playbook: common.yml
- import_playbook: db.yml
- import_playbook: php.yml
- import_playbook: web.yml
- import_playbook: lb.yml
```
In this example we know that PHP instances will come up only after DB but before WEB servers. Load Balancers will know necessary data about other hosts which is touched previously in play.  

If Your playbooks is written with idempotency in mind You can run them again and again when new instance appearing and this will safely add it to the cluster not disturbing others.  

In case independent hosts at scale You can switch to Puppet native Pull model with Ansible too.  
This require install Ansible on each host and instruct to monitor some repository once in a period.  

### Mixing declarative and imperative style

*main.yml*
```
- name: Installing foo package. Declarative style.
  package: 
    name: foo
    state: installed

- name: Check something with command which not changed state
  command: show something
  changed: False
  register: show_res

- name: Idempotency example for command module. Do not run if file is already there
  command: cat something.txt > /tmp/file.txt
  args:
    creates: /tmp/file.txt
```

## Ansible Inventory

Inventory is basement floor for every Ansible orchestration project.  
Never proceed until You will not have strong opinion what info from what sources do You require in inventory.  

From historical reasons explanation on official site start from single text file `hosts` in ini format.  
This is good for first time understanding. But when You already understand it is better to start using more advanced methods  

First of all instead of one file is better to point Ansible to whole folder containing inventory information  
See [Fedora Ansible repo](https://infrastructure.fedoraproject.org/cgit/ansible.git/tree/inventory) as example  
This come from Ansible best practices [alternative-directory-layout](http://docs.ansible.com/ansible/devel/user_guide/playbooks_best_practices.html#alternative-directory-layout)  

Try to use host can be in multiple groups. Please use groups as Labels in Kubernetes. 
Try to not use `[groupname:children]` this makes inventory cloud incompatible
> only exception is [group renaming](http://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html#static-groups-of-dynamic-groups) for cloud dynamic inventory.  

Try not put variables in `hosts` file this makes inventory cloud incompatible. Use `group_vars` directory.

Avoid using `host_vars` except `localhost` other host names can be altered in future.

When dealing with clouds always prefer use of [Dynamic Inventory](http://docs.ansible.com/ansible/intro_dynamic_inventory.). It is an axiom.   
Please read also about [using-inventory-directories-and-multiple-inventory-sources](http://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html#using-inventory-directories-and-multiple-inventory-sources) and [static-groups-of-dynamic-groups](http://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html#static-groups-of-dynamic-groups).  
This is powerful way how to make Your inventory really extendable.  

## Variables

Variables are atached to hosts even when You mention it in group it will boil down that You need target host from this group to access Variable
examples 
*cat group_vars/client/main.yml*
```
env_dns: "{{ hostvars[groups['ns'][0]]['ansible_host'] }}"
domain_name: "{{ hostvars[groups['ns'][0]]['bind_zone_name'] }}"
```
There is 2 kinds of Variables:  
### Variables
something which You know from Ansible project itself even before connect to any hosts 
To analyze this variables You can run such command:  
```
ansible all -m debug -a "var=hostvars[inventory_hostname]"
```
### Facts
something which You discover, declare, or attach during runtime
Example of automatically gathered facts.
```
ansible all -m setup
```
Variables and Facts is similar in usage but if there is choyse You should preffer Variables because Ansible know it already.

Variable files do not have reqirements on file name. So You can not name it `main.yml` and free Your imagination.

Variable names in role should include <role_name>_<variable_name> to avoid naming clash.

If You constantly overwrite some configuration parameter default value in role put default in `<role_path>/defaults/*` Your values in `<role_path>/vars/*`  
This will allow Your future self keep trace what is default without Your changes.


## Using Ansible with cloud solutions.

For best user experience put controller host more close to managed resources.  
Ansible has modules for spin up VMs and resources in AWS, OpenStack, Azure- try not to use them at all.  

VM and resources is more easily created and managed using cloud native provision solution like CloudFormation, Heat, Terraform which purpose is to manage not only creation but whole lifecycle of virtual infrastructure.  
Only after separate provision stage Ansible empowered with [Dynamic Inventory](http://docs.ansible.com/ansible/intro_dynamic_inventory.) should come into play.  

Using Ansible You can deploy different purpose VMs from start. But what to do if You need scale up or down and cloud solution not provide LBaaS or Your application logic is too tricky for simple Load Balancing algorithm?  

There is ready to use solution for this called [Ansible Tower](http://docs.ansible.com/ansible-tower/)  

With Ansible Tower You can use scenario with [call-home](http://cloudinit.readthedocs.io/en/latest/topics/examples.html#call-a-url-when-finished) url. When VM is ready for deployment after provision it can contact with Ansible to [Launch Job Template](http://docs.ansible.com/ansible-tower/latest/html/towerapi/launch_jobtemplate.html) from Userdata script Url call.  

Using call home url You can empower only Ansible Controller to make modifications and have access to all infrastructure.  

## Continuous Integration and Continuous Deployment with Ansible

Ansible playbooks can 
* deploy new version of code
* run tests
* deploy code to Production
  
Options:

### BitBucket( or Git repo without web interface) + Ansible Tower. 

Tower Templates can check repository time to time and react when something changed in particular branch and run predefined deployment scenario.  

### Jenkins + Ansible

Jenkins pipelines can launch Ansible playbooks. No experience with that.  

### Gitlab + Ansible in Docker container

Gitlab has own [CI/CD and runners concepts](https://docs.gitlab.com/ee/ci/quick_start/). This Runner can reside among managed host periodically watching for repository changes. One option is to put Docker container with Ansible to run deployment Job and integration tests Job.  

*Example pipeline:*
* Local tests with Docker containers (Smoke and Unit Tests)
* Provision Test environment (Cloudformation, Heat or Terraform)
* Deploy (Ansible)
* Integration Regresion UI Tests (Ansible as a launcher)
* Deprovision Test environment (Cloudformation, Heat or Terraform) *Optional*
* Deploy to Production (Ansible)
