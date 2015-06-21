# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative 'vagrant_rancheros_guest_plugin.rb'

# To enable rsync folder share change to false
$rsync_folder_disabled = false
$number_of_nodes = 1
$vm_mem = "1024"
$vb_gui = false

#download_docker_experimental = "https://github.com/dockervlan/dockervlan-vagrant/releases/tag/latest"
download_docker_experimental = ""


# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  config.vm.box   = "rancherio/rancheros"
  config.vm.box_version = ">=0.3.1"

  (1..$number_of_nodes).each do |i|
    hostname = "rancher-%02d" % i

    config.vm.define hostname do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.memory = $vm_mem
            vb.gui = $vb_gui
        end

        ip = "172.19.8.#{i+100}"
        node.vm.network "private_network", ip: ip

        # Disabling compression because OS X has an ancient version of rsync installed.
        # Add -z or remove rsync__args below if you have a newer version of rsync on your machine.
        node.vm.synced_folder ".", "/opt/rancher", type: "rsync",
            rsync__exclude: ".git/", rsync__args: ["--verbose", "--archive", "--delete", "--copy-links"],
            disabled: $rsync_folder_disabled

            if download_docker_experimental != ""
              perform_download_docker_experimental = <<-EOF
          	    echo 'Performing 10MB download of Docker experimental build'
          	    wget -q #{download_docker_experimental} -O /opt/bin/docker-1.8.0-dev
              EOF
          	else
              perform_download_docker_experimental = <<-EOF
          	    sudo cp /opt/rancher/docker-1.8.0-dev /opt/bin
              EOF
          	end

          	bootstrap_script = <<-EOF
          	  sudo mkdir /opt/bin
          	  #{perform_download_docker_experimental}
          	  sudo chmod +x /opt/bin/docker-1.8.0-dev

              cp /opt/rancher/custom-docker.yml /var/lib/rancher/conf/.
          	  sudo ros service enable /var/lib/rancher/conf/custom-docker.yml
          	  echo "Rebooting..."
              sudo reboot

            EOF
            node.vm.provision :shell, :inline => bootstrap_script
    end
  end
end
