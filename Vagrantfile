# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "geerlingguy/ubuntu2004"

  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.ssh.insert_key = false
  config.ssh.private_key_path = File.expand_path('~/.vagrant.d/insecure_private_key')

  config.vm.provider :virtualbox do |v|
    v.memory = 512
    v.linked_clone = true
  end

  {srv1: '192.168.56.10',
   srv2: '192.168.56.11',
   srv3: '192.168.56.12'}.each_pair do |srv_name, srv_ip_addr|
     config.vm.define srv_name do |srv|
       srv.vm.hostname = "#{srv_name}.test"
       srv.vm.network :private_network, ip: srv_ip_addr
     end
   end
end
