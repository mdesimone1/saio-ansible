SAIO Ansible playbook via Vagrant
=================================

This project assumes you have Virtualbox and Vagrant.

 * https://www.virtualbox.org/wiki/Downloads
 * http://www.vagrantup.com/downloads.html
 
- PS: I use VirtualBox 5.1.6 r110634 and Vagrant 1.8.5

Output Screenshot
 * https://gist.github.com/chianingwang/9fd5a4cf6e7f51af35bd831da6327530

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
 
- Double Check
 1. `swift stat -v`
 
- Back to Local
 1. `exit`
 
###Wipe Out
 1. `vagrant destroy`

