Vagrant.configure(2) do |config|
  config.vm.box = "bento/centos-7.2"
  #config.vm.box = "chef/fedora-20"
  #config.ssh.username = 'vagrant'
  #config.ssh.password = 'vagrant'
  #config.ssh.insert_key = 'false'
  config.ssh.forward_agent = 'true'
  #config.ssh.private_key_path = "/Users/johnnywa/gitlab/ansible-saio/.vagrant/machines/server0/virtualbox/id_rsa"
  config.ssh.private_key_path = "/Users/johnnywa/gitlab/saio-ansible/.vagrant/machines/server0/key/vagrant_id_rsa"
  VMS = 1
  (0..VMS-1).each do |vm|
    config.vm.define "server#{vm}" do |g|
        g.vm.hostname = "server#{vm}"
        g.vm.network :private_network, type: "dhcp"
        g.vm.provider :virtualbox do |vb|
            vb.memory = 2048
            vb.cpus = 2
        end

        if vm == (VMS-1)
            g.vm.provision :ansible do |ansible|
                ansible.playbook = "site.yml"
                ansible.groups = {
                    "storage" => (0..VMS-1).map { |v| "server#{v}" }
                }
                ansible.limit='all'
            end
        end
    end
  end
end
