# -*- mode: ruby -*-
# vi: set ft=ruby :
MACHINES = {
  
  :frontweb => {
      :box_name => "ubuntu/jammy64",
      :vm_name => "frontweb",
      :cpus => 1,
      :memory => 1024,
      :public => {:ip => "192.168.1.145", :adapter => 3},
      :net => [
                  ["192.168.56.145", 2, "255.255.255.0"],
              ]  
  },

  :backend1 => {
      :box_name => "ubuntu/jammy64",
      :vm_name => "backend1",
      :cpus => 1,
      :memory => 1024,
      :net => [
                  ["192.168.56.222", 2, "255.255.255.0"],
              ]  
  },

  :backend2 => {
      :box_name => "ubuntu/jammy64",
      :vm_name => "backend2",
      :cpus => 1,
      :memory => 1024,
      :net => [
                  ["192.168.56.212", 2, "255.255.255.0"],
              ]  
  },

  :mastersdb => {
      :box_name => "ubuntu/jammy64",
      :vm_name => "mastersdb",
      :cpus => 1,
      :memory => 1024,
      :net => [
                  ["192.168.56.69", 2, "255.255.255.0"],
              ]  
  },
  
  :slavesdb => {
      :box_name => "ubuntu/jammy64",
      :vm_name => "slavesdb",
      :cpus => 1,
      :memory => 1024,
      :net => [
                  ["192.168.56.98", 2, "255.255.255.0"],
              ]  
  },

  :monitoring => {
      :box_name => "ubuntu/jammy64",
      :vm_name => "monitoring",
      :cpus => 2,
      :memory => 4096,
      :net => [
                  ["192.168.56.60", 2, "255.255.255.0"],
              ]  
  },
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxconfig[:vm_name]

      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
      end
      
      boxconfig[:net].each do |ipconf|
        box.vm.network("private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2])
        #box.vm.network("public_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2], bridge: "wlp2s0")
      end

      if boxconfig.key?(:public)
        box.vm.network "public_network", **boxconfig[:public], bridge: "wlp2s0"
      end  

      box.vm.provision "shell", inline: <<-SHELL
        sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
        sudo systemctl restart sshd
      SHELL
    end
  end
end
