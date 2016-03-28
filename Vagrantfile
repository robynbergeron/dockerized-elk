Vagrant.configure(2) do |config|
  config.vm.box = "projectatomic/adb"

  # Setup a DHCP based private network
  config.vm.network "private_network", type: "dhcp"

  # Forward ports
  config.vm.network "forwarded_port", guest: 5000, host: 5000
  config.vm.network "forwarded_port", guest: 5601, host: 5601
  config.vm.network "forwarded_port", guest: 9200, host: 9200
  config.vm.network "forwarded_port", guest: 9300, host: 9300

  config.vm.provider "libvirt" do |libvirt, override|
    libvirt.driver = "kvm"
    libvirt.memory = 2048
    libvirt.cpus = 2
  end

  config.vm.provider "virtualbox" do |vbox, override|
    vbox.memory = 2048
    vbox.cpus = 2

    # Enable use of more than one virtual CPU in a virtual machine.
    vbox.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
#   Uncomment the line below if you want more output. Refer to Vagrant and/or Ansible docs if you want more info.
#   ansible.verbose = "vvv"
  end

end
