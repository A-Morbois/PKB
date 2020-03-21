# Devops_Project
tags = #Devops #Linux 


## pre-requisite 
>Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. 

Install additional tool on virtualbox: 
`sudo apt install virtualbox-guest-x11 virtualbox-guest-utils virtualbox-guest-dkms`



## Vagrant 

- `sudo apt-get-install Vagrant`
- `vagrant --version`

Each machine created on Vagrant has it's own directory
`vagrant init <Name>` -> will create a Vagrantfile (Name : box name on vagrant ex : ubuntu/focal64)

`vagrant up` -> create the machine
`vagrant ssh` -> access your machine
`vagrant reload` -> reload config after change in Vagrantfile
`vagrant suspend` -> pause/freeze the machine 
`vagrant resume` -> resume the machine 
`vagrant halt` -> shutdown
`vagrant destroy` -> reset the machine. Anything you've done on it will be lost (exept any file on the sahre directory)

vagrant create automatically a shared folder called "/vagrant" on the guest
Can be change in Vagrantfile
> config.vm.synced_folder "../data", "/vagrant_data" // can be disabled with ", disable : true"

3 types of network available (still in Vagrantfile):
- config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
- config.vm.network "public_network"
- config.vm.network "private_network", ip: "192.168.33.10"

Provisionning : execute a script at the end of the boot of the machine
Can be a script or a configuration manager (chef, puppet etc)

run provisionning at the creation or need to run it manually `vagrant provision`
`config.vm.provision "shell", path "<Bash_Script_path>"`

VagrantFile is written in ruby