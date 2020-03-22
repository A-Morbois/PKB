# Configuration_Management
tags = #Devops #Linux #Ansible

## Change Management
- Every change need to be documented, we need the history of everything that happens to retrace actions.
- Idempotent. Run a operation several time with the same result each time. -> Make sure that the configuration is always the same.

>Hard to do with a shell Script

## What is Ansible

- Similar as Chef Puppet or Slat
- Agentless
- use SSH to connect client-target
- Use python for scripting

//pip is the package installer for pyhton
//brew is the package installer for macOS

## Install Ansible

### Ubunbtu
- Create a Vagrant machine
- `antoine@CYWY:~$vagrant ssh`
- `vagrant@ubuntu-focal:~$ sudo apt-add-repository ppa:ansible/ansible`
- `sudo apt-get update`
- `sudo apt-get install ansible`

### Centos
- ``antoine@CYWY:~$vagrant ssh``
- `[vagrant@localhost ~]$ sudo yum install epel-release`
- `[vagrant@localhost ~]$ sudo yum install ansible`

### Python
- `sudo easy install pip`
- `pip install ansible`

### Usage

`/etc/ansible/host` : *Configuration de l'host*
You can group machine / use pattern or individual

If not good, change the vagrant config to setup a private network to make some tests ([[202003211551]] Vagrant)
using vagrantfile and config.vm.network "private_network".
Vagrant reload


// Side notes :
IP ubuntu : "192.168.33.20"
IP Centos : "192.168.33.10"

Inside /etc/ansible/host : we can add IP or Host (referenced into /etc/host)


`ansible all -a "cat /etc/hosts"` : discover if all machine answer your ansible configuration

### set up SSH connection
* [[202003221000]] setu_SSH_Connection


In `/etc/hosts` add the reference to the other machine `192.168.33.20 ubuntu` so it's easier to switch and know your network

ansible ubuntu -a "bash --version"

### Yaml Playbook Modules

! Careful with indentation / no tabs
  
- **hosts**: 
  tasks:

- **become:** yes 
> use sudo when executing a task

-  `apt:` : the package manager from the system (would be yum for centos for exemple)

- **state:** present/absent`

- service:
	name: apache2
	**state:** started/stopped/restarted 
	**enabled:** yes 
> get start on boot

- yum: **name={{ item }}** state=present
      **with_items:
        - mod_php7lw
        - php7lw_cli** 
> loop through item to install all of them


- **file :**
     path:
     owner:
     group:
>Change the owner of a file

- **lineinfile**:
    path:
    regexp:
    line: 
    state: present
>Change the content of a line

- **lineinfile**:
    path:
    insertafer: 
    line:  ... |
    ... 

- **notify:**
	- 'xyz'

// at the beginning of the file
	-> handlers:
		- name : xyz
		  service: do something

- **copy:**
    src:
    dest:
    owner:
    group:
    mode: 
>copy a file

- **mysql_user** //need MySQL-python package
	 name :
	 password:  // set the password for the root
	 state:
	 //login_password
	 //login_user


- **mysql_db:**
     name: <db_name>
     state: present 
>create a database // make sur it is present

- **mysql_user:**
	name: 
	password:
	host: '%' // all
	priv: <db_name>.*:ALL
	state: present
>create a specific user with full access on db_name


- **mysql_user:**
	name: '' 
	host_all: yes
	state: asbent
>remove anonymous account


### Exemple of Yaml playblook
*Make sure the host ubuntu have apache installed and it should started when booting*

```
- hosts: ubuntu
  become: yes
  tasks:
    - name: Ensure that Apache is installed 
      apt:
        name: apache2
        state: present

    - name: Ensure that apache is running and will be started on boot
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Install PHP 7 and required packages
      yum: name={{ item }} state=present
      with_items:
        - mod_php7lw
        - php7lw_cli
        - ...

 ```


