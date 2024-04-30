# -*- mode: ruby -*-
# vi: set ft=ruby :

#
# Load config
# 
require 'yaml'
#setup = YAML.load_file('_aux/config.yml')
#setup['external_ip'] = '172.21.1.2' unless setup['external_ip']

Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  def is_arm64()
    # `uname -m` == "arm64" || `/usr/bin/arch -64 sh -c "sysctl -in sysctl.proc_translated"`.strip() == "0"
    `uname -m`.strip() == "arm64"
  end
 
  if is_arm64()
    #config.vm.box = "opensuse/Leap-15.5.aarch64"
    config.vm.box = "bento/ubuntu-24.04"
    # config.vm.box = "focal-server2"
    provider = "vmware_fusion"
  else
    # config.vm.box = "opensuse/Leap-15.5.x86_64"
    config.vm.box = "bento/ubuntu-24.04"
    # provider = "virtualbox"
    provider = "vmware_fusion"
  end

  config.vm.box_check_update = false
  config.vm.hostname = "vvroom"
  #config.vm.network :private_network

  config.vm.provider provider do |prov|
    if provider == "vmware_fusion"
      prov.allowlist_verified = true
      #prov.vmx["ethernet1.virutalDev"] = "vmxnet3"
      prov.vmx["ethernet0.virtualDev"] = "vmxnet3"
      #prov.vmx["ethernet0.generatedAddress"] = "00:0C:29:00:31:9D"
      prov.vmx["ide1:0.present"] = "TRUE"
      prov.vmx["ide1:0.deviceType"] = "cdrom-raw"
      prov.vmx["memsize"] = "4096"
      prov.vmx["numvcpus"] = "4"
    
    elsif provider == "virtualbox"
      # might be a bug with this adding dup interfaces on multiple vagrant up or provision invocations
      #config.vm.network "private_network", ip: setup['external_ip']
      prov.customize ["modifyvm", :id, "--vram", "32"]
      prov.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
      # need a cdrom to mount the extensions ISO
      prov.name = "vvroom"
      system("VBoxManage storagectl vvroom --name IDE --remove 2>&1")
      system("VBoxManage storagectl vvroom --name IDE --add ide 2>&1")
      system("VBoxManage storageattach vvroom --storagectl IDE --port 0 --device 0 --type dvddrive --medium emptydrive 2>&1")

    end

    # Display the VirtualBox GUI when booting the machine
    # uncomment this when adding the cdrom to inject the extensions ISO for mounting/install of tools
    prov.gui = true

    # Customize the cpu count and memory on the VM:
    prov.memory = "4096"
    prov.cpus = 4
  end

  #config.vm.synced_folder "./provisioning", "/vagrant/provisioning"
  #config.vm.synced_folder "./server", "/vagrant/server"
  #config.vm.synced_folder "./web", "/vagrant/web"

  # View the documentation for the provider you are using for more
  # information on available options.

  # Change link to python3 and install pip3 
  config.vm.provision "Python version change", before: :all, type: "shell", inline: <<-SHELL
    osid=$(grep '^ID=' /etc/os-release | cut -d'=' -f2 | tr -d '"')
    case $osid in
      ubuntu)
        while fuser /var/lib/dpkg/lock >/dev/null 2>&1; do sleep 1; done;
        apt-get update
        apt-get install -y python3-pip && echo Pip3 installation complete.
        ln -f /usr/bin/python3 /usr/bin/python && echo Python default version was changed to 3.
        pip install --break-system-packages --upgrade ansible yamllint
        ;;
      opensuse-leap)
        zypper ref
        zypper -n update -y
        zypper in -y python311-pip python311
        pip install --upgrade ansible-base ansible-core yamllint
        ;;
    esac
    
  SHELL
      
  #
  # Run Ansible from the Vagrant Host
  #
  config.vm.provision "ansible_local" do |ansible|
    ansible.verbose = "false"
    ansible.install = "false"
    ansible.install_mode = "pip"
    ansible.limit = 'all,localhost'
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "_aux/vvroom-ansible-local.yml"
    ansible.galaxy_role_file = "_aux/galaxy.yml"
    # ansible.galaxy_roles_path = ".ansible/collections"
    ansible.galaxy_command = "ansible-galaxy collection install -r %{role_file}" # -p %{roles_path}"
  end

	config.vm.post_up_message = [
	  "",
	  "##########################################################################",
	  "#                                                                        #",
	  "#    Your development VM is now up and running                           #",
	  "#                                                                        #",
	  "##########################################################################",
    "",
    "  Don't forget to mount vmtools with the CD and install on first launch.",
    "",
	  "",
	  "  Start the server",
    "",
    "  'cd /vagrant'",
    "  influx:  do some stuff",
    "",
    "  Server: 'cd /server' and ",
    "  'go run main.go'",
    "",
  ].join("\n")	 

end

