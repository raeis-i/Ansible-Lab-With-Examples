# Ansible_Lab_With_Real_Examples
In the previous repository, we learned how to bring up an Ansible lab and run a simple ad-hoc command.

[Full-Automated-Ansible-Lab-with-Vagrant-Virtualbox](https://github.com/raeis-i/Full-Automated-Ansible-Lab-with-Vagrant-Virtualbox).

In this repository, we are going to use:

1- ["ansible.builtin.user"](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html) module, to manage user accounts.

2- ["ansible.builtin.apt" ](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html) module, to install and uninstall Nginx.

3- ["ansible.builtin.service" ](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html) module, to start and enable Nginx services.

You can find builtin modules [here](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html).


## Installation
```bash
$ git clone https://github.com/raeis-i/Ansible-Lab-With-Examples.git
$ cd Ansible_Lab_With_Examples
$ vagrant up
$ vagrant ssh node1
$ vagrant@node1:~$ bash ansible.sh
$ vagrant@node1:~$ cd ansible/
```
## Encrypt variables and create user in all nodes
```bash
vagrant@node1:~/ansible$ ansible-vault encrypt playbooks/user_vars.yml 
New Vault password: 
Confirm New Vault password:
Encryption successful
vagrant@node1:~/ansible$ ansible-playbook -i inventory.ini playbooks/create_user.yml --ask-vault-pass
Vault password: 

PLAY [Create a user on all servers] **************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************
ok: [node2]
ok: [node3]
ok: [node1]
ok: [node5]
ok: [node4]

TASK [Ensure the user is created] ****************************************************************************************************************
[WARNING]: The input password appears not to have been hashed. The 'password' argument must be encrypted for this module to work properly.
changed: [node4]
changed: [node1]
changed: [node2]
changed: [node3]
changed: [node5]

PLAY RECAP ***************************************************************************************************************************************node1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node2                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node3                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node4                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node5                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

vagrant@node1:~/ansible$
vagrant@node1:~/ansible$ cat /etc/passwd | grep -i newuser
newuser:x:1002:1002::/home/newuser:/bin/sh
vagrant@node1:~/ansible$ 
```
## Remove user in nodes 1 and 2
You can also run the playbook for specific hosts or groups directly from the command line using the -l (limit) option.

By directly passing the username as an extra variable (-e "username=newuser"), Ansible will use this value during playbook execution.
```bash
vagrant@node1:~/ansible$ ansible-playbook -i inventory.ini playbooks/remove_user.yml -l node1,node2 -e "username=newuser"

PLAY [Create a user on all servers] **************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************
ok: [node2]
ok: [node1]

TASK [Ensure the user is created] ****************************************************************************************************************
changed: [node2]
changed: [node1]

PLAY RECAP ***************************************************************************************************************************************
node1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node2                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

vagrant@node1:~/ansible$ cat /etc/passwd | grep -i newuser
vagrant@node1:~/ansible$
```
You can install,uninstall Nginx via below commands, try to install Nginx in all servers and check it.
```bash
vagrant@node1:~/ansible$ ansible-playbook -i inventory.ini playbooks/install_nginx.yml
vagrant@node1:~/ansible$ ansible-playbook -i inventory.ini playbooks/uninstall_nginx.yml
```

## Introduction to each file
1-Vagrantfile : 
Configurations related to the VM are stored in this file, you can change CPU and Ram.

2-bootstrap.sh : Change SSH config in all nodes and Add all nodes to hosts file in all servers.

3-vars.rb : You can use image name or download it, and define the number of total nodes and Network range.

4- ansible.sh : This file will copy only to node1 as an Ansible node to manage other nodes. This file will install Ansible, generate SSH key ,copy Key to all nodes, and create an inventory file.

5-./ansible/playbooks/user_vars.yml :
```bash
username: "newuser" #Define the username of the user to be created.
user_password: "newuser" #Define password
```

6-./ansible/playbooks/create_user.yml
```bash
---
- name: Create a user on all servers
  hosts: all  #Specifies that the playbook should run on all hosts defined in the inventory.
  become: yes #Executes tasks with elevated privileges (as the root user) using sudo.
  vars_files: #Includes the user_vars.yml file to load variables.
    - user_vars.yml #Looks for this file in the current dir

  tasks:
    - name: Ensure the user is created
      ansible.builtin.user: #Ansible's built-in module for managing user accounts.
        name: "{{ username }}" #The username of the user to be created, taken from the username variable.
        password: "{{ user_password }}" #The password or password hash of the user, taken from the user_password variable.
        state: present #In Ansible’s user module ensures the specified user exists. If the user is missing, it will be created. If the user exists, their properties (like password, shell, etc.) will be updated to match the playbook.
```


7-./ansible/playbooks/remove_user.yml 
```bash
---
- name: Remove user on all servers
  hosts: all  #Specifies that the playbook should run on all hosts defined in the inventory.
  become: yes #Executes tasks with elevated privileges (as the root user) using sudo.

  tasks:
    - name: Ensure the user is removed
      ansible.builtin.user: #Ansible's built-in module for managing user accounts.
        name: "{{ username }}" #The username of the user to be removed, taken from the username variable.
        state: absent #Absent would ensure that the user account does not exist on the target hosts. If the user exists, it will be removed
```
8-./ansible/playbooks/install_nginx.yml
```bash
---
- name: Install Nginx on all servers
  hosts: all
  become: yes
  tasks:
    - name: Update apt cache 
      ansible.builtin.apt:
        update_cache: yes #Updates the apt package index on the target hosts.

    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present #Installs the Nginx package using the apt package manager.

    - name: Ensure Nginx is running and enabled
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes #Starts the Nginx service and ensures that it is set to start automatically during system boot.
```
9-./ansible/playbooks/uninstall_nginx.yml
```bash
---
- name: Uninstall Nginx on all servers
  hosts: all
  become: yes
  tasks:
    - name: Uninstall Nginx
      ansible.builtin.apt:
        name: nginx
        state: absent #Uninstalls the Nginx package using the apt package manager.



```


## Contributing
Pull requests are welcomed. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update the tests as appropriate.

## License

[MIT](https://choosealicense.com/licenses/mit/)
