# setu_SSH_Connection
tags = #SSH #Linux


`machine1$ ssh-keygen`
`cat <path_public_key>`

`ssh machine2`
`machine2$ vim .ssh/authorized_keys`
>paste the public key


-> no more need for the password after that


// ctrl+D to logout the ssh


## Troubleshooting 

### With Vagrant
using vagrant in the same Vagrantfile, the ssh keygen will be overwrite from one VM to another as the Vagrant directory is shared.
[Here](https://github.com/hashicorp/vagrant/issues/10280)
