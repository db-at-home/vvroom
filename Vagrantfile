# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
#setup = YAML.load_file('_aux/config.yml')
#setup['external_ip'] = '172.21.1.2' unless setup['external_ip']

Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # get arch as we might be limited to a specific provider based on it
  def is_arm64()
    # `uname -m` == "arm64" || `/usr/bin/arch -64 sh -c "sysctl -in sysctl.proc_translated"`.strip() == "0"
    `uname -m`.strip() == "arm64"
  end

  # decide on provider and box to use (as again, that can be restrictive)
  if is_arm64()
    #config.vm.box = "opensuse/Leap-15.5.aarch64"
    config.vm.box = "bento/opensuse-leap-15.6"
    #config.vm.box = "bento/ubuntu-24.04"
    # config.vm.box = "focal-server2"
    provider = "vmware_fusion"
  else
    config.vm.box = "opensuse/Leap-15.6.x86_64"
    #config.vm.box = "bento/ubuntu-24.04"
    provider = "virtualbox"
    # provider = "libvirt"    
    # provider = "vmware_fusion"
  end

  # do other non-provider specific vm config
  config.vm.hostname = "vvroom"
  #config.vm.disk :disk, name: "devbox", size: "30GB"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.56.12"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Disable the default share of the current code directory. Doing this
  # provides improved isolation between the vagrant box and your host
  # by making sure your Vagrantfile isn't accessible to the vagrant box.
  # If you use this you may want to enable additional shared subfolders as
  # shown above.
  # config.vm.synced_folder ".", "/vagrant", disabled: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.

  # View the documentation for the provider you are using for more
  # information on available options.
  config.vm.provider provider do |prov|

    # Display the VirtualBox GUI when booting the machine
    prov.gui = true

    # Customize the cpu count and memory on the VM:
    prov.memory = "4096"
    prov.cpus = 4

    if provider == "vmware_fusion"
      # check that box is properly configured as vmware guest
      prov.allowlist_verified = true

      prov.vmx["ethernet0.virtualDev"] = "vmxnet3"
      prov.vmx["ethernet1.virutalDev"] = "vmxnet3"
      prov.vmx["ide1:0.present"] = "TRUE"
      prov.vmx["ide1:0.deviceType"] = "cdrom-raw"
      prov.vmx["memsize"] = "4096" # generic customize does nothing
      prov.vmx["numvcpus"] = "4"   # generic customize does nothing
      prov.vmx["vhv.enable"] = "TRUE"
      prov.vmx["vvtd.enable"] = "TRUE"

    elsif provider == "virtualbox"
      # might be a bug with this adding dup interfaces on multiple vagrant up or provision invocations
      prov.customize ["modifyvm", :id, "--vram", "32"]
      prov.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
      # might need a cdrom to mount the extensions ISO
      # prov.name = "vvroom"
      # system("VBoxManage storagectl vvroom --name IDE --remove 2>&1")
      # system("VBoxManage storagectl vvroom --name IDE --add ide 2>&1")
      # system("VBoxManage storageattach vvroom --storagectl IDE --port 0 --device 0 --type dvddrive --medium emptydrive 2>&1")

    end
  end

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
        zypper -n up -y
        zypper in -y python311-pip python311
        pip3 install --upgrade ansible ansible-base ansible-core ansible-lint yamllint
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
    ansible.compatibility_mode = "2.0" # breaks with ansible version >= 10
    ansible.playbook = "provisioning/vagrant/ansible-local.yml"
    ansible.galaxy_role_file = "provisioning/vagrant/requirements.yml"
  # ansible.galaxy_roles_path = ".ansible/collections"
    ansible.galaxy_command = "ansible-galaxy collection install -r %{role_file}" # -p %{roles_path}"
  end

	config.vm.post_up_message = [
    "",
    "##########################################################################",
    "#                                                                        #",
    "#              Your development VM is now up and running                 #",
    "#                                                                        #",
    "##########################################################################",
    "",
    "# To get a prompt on the VM",
    "",
    "  host:~> vagrant ssh",
    "",
    "# For vmware provider, confirm your guest has latest open-vm-tools.",
    "# Similarly, for virtualbox provider, check virtualbox-guest-tools.",
    "",
    "# The postgresql db container is cli accessible",
    "",
    "  vvroom:~> psql -U postgres -h pghost pgdb",
    "",
    "##########################################################################",
    "",
  ].join("\n")	 

end

