# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  config.vm.box = "fedora/34-cloud-base" 

  config.vm.provider "virtualbox" do |vb|
    vb.name    = 'fedora-docker-webgoat-juiceshop'
    vb.memory  = 2048
    vb.cpus    = 2
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 9090, host: 9090
  config.vm.network "forwarded_port", guest: 8888, host: 8888
  config.vm.network "forwarded_port", guest: 3000, host: 3000

  $script = <<-SCRIPT

  # Sleep is necessary because the network is not available right after the start of the VM. All following actions
  # depend heavilly on the network.

  sleep 45

  rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-34-primary
  dnf upgrade -y

  dnf -y install dnf-plugins-core
  dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
  dnf install -y docker-ce docker-ce-cli containerd.io

  systemctl enable docker
  systemctl start docker
 
  # Synchronize time
  dnf install -y ntpsec
  systemctl enable ntpd
  systemctl start ntpd
  timedatectl set-timezone Europe/Amsterdam

  # Allow the default Vagrant user to use Docker  
  usermod -aG docker vagrant

  # Start the training tools
  docker run -d -p 8080:8080 -p 9090:9090 -p 80:8888 -e TZ=Europe/Amsterdam webgoat/goatandwolf:latest
  docker run -d -p 3000:3000 bkimminich/juice-shop:latest

  SCRIPT
  
  config.vm.provision "shell",
      inline: $script
	  
end
