# -*- mode: ruby -*-
# vi:set ft=ruby sw=2 ts=2 sts=2:

NUM_MASTER_NODE = 1
NUM_WORKER_NODE = 1

IP_NW = "192.168.56."
MASTER_IP_START = 1
NODE_IP_START = 2
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  # Disable automatic box update checking.
  config.vm.box_check_update = false
  config.vm.provision "shell", path: "ubuntu/initialSetup.sh"

  

  # Provision Master Nodes
  (1..NUM_MASTER_NODE).each do |i|
      config.vm.define "master0#{i}" do |node|
        node.vm.synced_folder ".", "/vagrant/"
        node.vm.provider "virtualbox" do |vb|
            vb.name = "master0#{i}"
            vb.memory = 2048
            vb.cpus = 2
        end
        node.vm.hostname = "master0#{i}"
        node.vm.network :private_network, ip: IP_NW + "#{MASTER_IP_START + i}"
        node.vm.network "forwarded_port", guest: 22, host: "#{2710 + i}"
        node.vm.provision "setup-hosts", :type => "shell", :path => "ubuntu/setup-hosts.sh" do |s|
          s.args = ["enp0s8"]
        end
        node.vm.provision "setup-dns", type: "shell", :path => "ubuntu/update-dns.sh"
        node.vm.provision "shell", type: "shell", path: "ubuntu/setupmaster.sh"
      end
  end

  # Provision Worker Nodes
  (1..NUM_WORKER_NODE).each do |i|
    config.vm.define "worker0#{i}" do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.name = "worker0#{i}"
            vb.memory = 1024
            vb.cpus = 2
        end
        node.vm.hostname = "worker0#{i}"
        node.vm.network :private_network, ip: IP_NW + "#{NODE_IP_START + i}"
        node.vm.network "forwarded_port", guest: 22, host: "#{2720 + i}"
        node.vm.provision "setup-hosts", :type => "shell", :path => "ubuntu/setup-hosts.sh" do |s|
          s.args = ["enp0s8"]
        end
        node.vm.provision "setup-dns", type: "shell", :path => "ubuntu/update-dns.sh"
        node.vm.provision "setup-dns", type: "shell", :path => "ubuntu/setupworker.sh"
    end
  end

  # Run shell commands
  config.vm.provision "shell", inline: <<-SHELL
    # Set pass 123 with root account and allow SSH
    echo "123" | passwd --stdin root
    sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
    systemctl reload sshd
# Write the following content to file /etc/hosts to access machines according to HOSTNAME
cat >>/etc/hosts<<EOF
192.168.56.2 master01
192.168.56.3 worker01
192.168.56.4 worker02
EOF

  SHELL
end
