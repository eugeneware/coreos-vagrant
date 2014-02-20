# -*- mode: ruby -*-
# # vi: set ft=ruby :

# By default the coreos docker server will be set to the IP address below
# If you set the environmenet variable DOCKER_HOST=172.12.8.150 then
# all your docker commands will be against this server
PRIVATE_NETWORK = ENV['PRIVATE_NETWORK'] || "172.12.8.150"

Vagrant.configure("2") do |config|
  config.vm.box = "coreos"
  config.vm.box_url = "http://storage.core-os.net/coreos/amd64-generic/dev-channel/coreos_production_vagrant.box"

  # Expose this coreos server on the following private IP
  config.vm.network "private_network", ip: PRIVATE_NETWORK

  # Uncomment below to enable NFS for sharing the host machine into the coreos-vagrant VM.
  # config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true,  :mount_options   => ['nolock,vers=3,udp']

  # Expose docker on port 4243
  config.vm.provision "file" do |f|
    f.source = './docker-local.service'
    f.destination = '/tmp/docker-local.service'
  end

  # Stop the chaos monkey reboots
  config.vm.provision "file" do |f|
    f.source = './stop-reboot-manager.service'
    f.destination = '/tmp/stop-reboot-manager.service'
  end

$script = <<EOF
sudo cp /tmp/*.service /media/state/units/
sudo systemctl enable --runtime /media/state/units/stop-reboot-manager.service
sudo systemctl start stop-reboot-manager
sudo systemctl enable --runtime /media/state/units/docker-local.service
sudo systemctl start docker-local
EOF

  config.vm.provision :shell, :inline => $script

  # Fix docker not being able to resolve private registry in VirtualBox
  config.vm.provider :virtualbox do |vb, override|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  config.vm.provider :vmware_fusion do |vb, override|
    override.vm.box_url = "http://storage.core-os.net/coreos/amd64-generic/dev-channel/coreos_production_vagrant_vmware_fusion.box"
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end
end
