# -*- mode: ruby -*-
# vi: set ft=ruby :
VAGRANTFILE_API_VERSION = "2"

# Size of the cluster created by Vagrant
num_instances=3
# Change basename of the VM
instance_name_prefix="k8sn"
GATEWAY="192.168.16.1"
VM_NETMASK = "255.255.255.0"
VM_BRIDGE = ENV["VAGRANT_BRIDGE"] || "Intel(R) Dual Band Wireless-AC 3165"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = false     
  config.vm.box = "centos/7"
  config.vm.synced_folder ".", "/vagrant", disabled: true 
    
  config.vm.provider :virtualbox do |v|
    v.check_guest_additions = false
    v.memory = 2048 
    v.cpus = 1
    v.functional_vboxsf     = false
	v.customize ["modifyvm", :id, "--nictype1", "virtio"]
    v.customize ["modifyvm", :id, "--nictype2", "virtio"]
  end

  # Set up each box
  (1..num_instances).each do |i|
          
	  if i == 1
      vm_name = "k8sm-01"
    else      
	  vm_name = "%s-%02d" % [instance_name_prefix, i-1]
    end
		
   		
    config.vm.define vm_name do |host|
		host.vm.hostname = vm_name	
		ip = "192.168.16.#{i+210}"		
		host.vm.network "public_network", bridge: VM_BRIDGE, ip: ip, :auto_config => "false", :netmask => VM_NETMASK

	   host.vm.provision :shell, :inline => "echo '127.0.0.1\tlocalhost' > /etc/hosts", :privileged => true
	   host.vm.provision :shell, :inline => "[ -d /root/.ssh ] || mkdir -p -m 700 /root/.ssh ", :privileged => true
       host.vm.provision :file, source: "~/.ssh/authorized_keys ", destination: "/tmp/authorized_keys"
	   host.vm.provision :shell, :inline => "[ -f  /tmp/authorized_keys ] && mv /tmp/authorized_keys /root/.ssh/authorized_keys", :privileged => true
	   host.vm.provision :shell, :inline => "[ -f /root/.ssh/authorized_keys ] && chown root:root /root/.ssh/authorized_keys", :privileged => true
	   host.vm.provision :shell, :inline => "[ -f /root/.ssh/authorized_keys ] && chmod 600 /root/.ssh/authorized_keys", :privileged => true
	   host.vm.provision :shell, :inline => "setenforce 0", :privileged => true
       host.vm.provision :shell, :inline => "sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/sysconfig/selinux", :privileged => true
	   host.vm.provision :shell, path: File.join(File.dirname(__FILE__), "scripts/script.sh"), args: [GATEWAY]
	  	  
    end
  end
end
