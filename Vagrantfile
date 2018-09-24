# -*- mode: ruby -*-
# vi: set ft=ruby :

# Size of the cluster
num_instances=2

# IP variables for setting up the scripts to run on each node
node_ip_nw="172.18.18."
master_ip=node_ip_nw + "101"
pod_nw_cidr="10.244.0.0/16"

#Generate new using 'kubeadm token generate'
kubetoken = "b029ee.968a33e8d8e6bb0d"

$nodescript = <<NODESCRIPT
kubeadm join --token #{kubetoken} --discovery-token-unsafe-skip-ca-verification #{master_ip}:6443
NODESCRIPT

$masterscript = <<MASTERSCRIPT
kubeadm init --apiserver-advertise-address=#{master_ip} --pod-network-cidr=#{pod_nw_cidr} --token #{kubetoken} --token-ttl 0
mkdir -p /home/ubuntu/.kube
sudo cp -Rf /etc/kubernetes/admin.conf /home/ubuntu/.kube/config
sudo chown $(id -u ubuntu):$(id -g ubuntu) /home/ubuntu/.kube/config
MASTERSCRIPT

# Basename of the VM
instance_name_prefix="n"

Vagrant.configure("2") do |config|
  config.ssh.insert_key = true

  config.vm.box = "ubuntu/xenial64"
  config.vm.provision :shell, :path => "install.sh"

  # Set the Memory and CPU
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end

  # Don't attempt to update Virtualbox Guest Additions (requires gcc)
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  # Set up each box
  (1..num_instances).each do |i|
    if i == 1
      vm_name = "m"
    else
      vm_name = "%s%01d" % [instance_name_prefix, i-1]
    end

    config.vm.define vm_name do |host|
      host.vm.hostname = vm_name
      host.vm.synced_folder ".", "/vagrant"

      ip = node_ip_nw + "#{i+100}"
      host.vm.network :private_network, ip: ip
      # Set these variables to customize the cpu and memory size for certain nodes
      #host.vm.provider :virtualbox do |vb|
      #  vb.customize ["modifyvm", :id, "--cpus", "2"]
      #  vb.customize ["modifyvm", :id, "--memory", "4096"]

      if i == 1
        host.vm.provision :shell, :inline => $masterscript
      else
        host.vm.provision :shell, :inline => $nodescript
      end

    end
  end
end
