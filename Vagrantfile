$script = <<-SCRIPT
dd if=/dev/zero of=/dev/sda bs=1M count=10
reboot
SCRIPT

# Describe VMs
MACHINES = {
  :"PXEServer" => {
              # VM box
              :box_name => "generic/debian11",
              # VM CPU count
              :cpus => 1,
              # VM RAM size (Mb)
              :memory => 1024,
              # networks
              :net => [{ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "pxenet"}],
              # forwarded ports
              :forwarded_port => [],
              # configure script
              :configurer => "pxeserver.yml"
  },
  :"PXEClient" => {
              # VM box
              :box_name => "generic/debian11",
              # VM CPU count
              :cpus => 1,
              # VM RAM size (Mb)
              :memory => 1024,
              # networks
              :net => [{ip: '192.168.0.3', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "pxenet"}],
              # forwarded ports
              :forwarded_port => [],
              # configure script
              :configurer => ""
  }
}

Vagrant.configure("2") do |config|
  # Disable shared folders
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.box_check_update = false

  MACHINES.each do |boxname, boxconfig|
    # Apply VM config
    config.vm.define boxname do |box|
      # Set VM base box and hostname
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      # Additional network config if present
      if boxconfig.key?(:net)
        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
      end
      # Port-forward config if present
      if boxconfig.key?(:forwarded_port)
        boxconfig[:forwarded_port].each do |port|
          box.vm.network "forwarded_port", guest: 80, host: port
        end
      end
      # VM resources config
      box.vm.provider "virtualbox" do |v|
        # Set VM RAM size and CPU count
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
	if boxname.to_s ==  "PXEClient"
	  v.customize ['modifyvm', :id, '--boot1', 'disk', '--boot2', 'net', '--nicbootprio2', '1']
	  box.vm.provision 'shell', inline: $script
	end
      end

      if boxconfig[:configurer].length > 0
      box.vm.provision "ansible" do |ansible|
        ansible.playbook = boxconfig[:configurer]
        ansible.verbose = "v"
      end
      end
    end
  end
end

