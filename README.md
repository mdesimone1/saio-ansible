SAIO Ansible playbook
=====================
###Boot Cent7 and Build SAIO 
 1. `vagrant up`

###SSH to the Box
 1. `vagrant ssh`

###Test Swift
 1. `swift stat -v`
 1. `echo 'this is test' > test.txt`
 1. `swift upload test_container test.txt`
 1. `swift list `
 1. `swift list test_container`
####double check
 1. `swift stat -v`

###Wipe Out
####Back to Local
 1. `exit`
####Delete all 
 1. `vagrant destroy`

