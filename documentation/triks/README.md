# Triks


## Python Venv

There is some rumors that it is imposible to run ansible with module which have other Python library reqirements and not that library is not installed in system Python.  
There is workaround- You can install reqirement in python Venv. After this You should instruct Ansible use different Python location  

*group_vars/all/main.yml*
```
ansible_python_interpreter: "python" # Default absolute path '/usr/bin/python'
```

*site.yml*
```
---
- hosts: all
  gather_facts: True
  environment:
    PATH: "{{ ansible_env.HOME }}/{{ venv_name }}/bin:{{ ansible_env.PATH }}"
```


## From Yaml to Python trough Jinja2

Somethimes You need advanced logic but goal is too small for ansible `filter` plugin or ansible `module`.   
There is example how can we reach Python from inside task  


```
- name: Create filtered list according to facts on hosts
  set_fact:
    member_list: |
      {% set mem_list = [] %}
      {% for host in ansible_play_hosts %}
      {% if 'is_something' in hostvars[host] %}
      {% if hostvars[host]['is_something'] == False %}
      {% set _ = mem_list.append(hostvars[host]['ansible_fqdn']) %}
      {% endif %}
      {% endif %}
      {% endfor %}
      {{ mem_list }}
```


You can do to same also in [Jinja2 template](../../roles/bind/templates/bind_zone.j2)  
Thanks to this [ansver](https://stackoverflow.com/questions/35605603/using-ansible-set-fact-to-create-a-dictionary-from-register-results) for inspiration.


