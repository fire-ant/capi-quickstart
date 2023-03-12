# -*- mode: ruby -*-
# vi: set ft=ruby :

# Copyright The flintlock Authors

# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at

#     http://www.apache.org/licenses/LICENSE-2.0

# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"

  config.ssh.username = 'vagrant'
  config.ssh.password = 'vagrant'
  config.ssh.insert_key = false
  config.ssh.forward_agent = true
  config.vm.synced_folder "./k3s", "/home/vagrant", create: true

  config.vm.network "forwarded_port", guest: 9000, host: 9000
  config.vm.network "forwarded_port", guest: 6443, host: 6443
  config.vm.network "forwarded_port", guest: 7443, host: 7443
  config.vm.network "forwarded_port", guest: 22, host: 2222

  cpus = 4
  memory = 8192
  config.vm.provider :virtualbox do |v|
    # Enable nested virtualisation in VBox
    v.customize ["modifyvm", :id, "--nested-hw-virt", "on"]

    v.cpus = cpus
    v.memory = memory
  end
  config.vm.provider :libvirt do |v, override|
    # If you want to use a different storage pool.
    # v.storage_pool_name = "vagrant"
    v.cpus = cpus
    v.memory = memory
    override.vm.synced_folder "./", "/home/vagrant/flintlock", type: "nfs"
  end

  config.vm.provision "upgrade-packages", type: "shell", run: "once" do |sh|
    sh.inline = <<~SHELL
      #!/usr/bin/env bash
      set -eux -o pipefail
      apt-get update && apt-get upgrade -y
    SHELL
  end

  config.vm.provision "install-kvm", type: "shell", run: "once" do |sh|
    sh.inline = <<~SHELL
      #!/usr/bin/env bash
      set -eux -o pipefail
      sudo apt-get -y install qemu qemu-kvm libvirt-clients libvirt-daemon-system virtinst bridge-utils
      adduser 'vagrant' libvirt
      adduser 'vagrant' kvm
      systemctl enable libvirtd
      systemctl start libvirtd
    SHELL
  end

  config.vm.provision "install-containerd", type: "shell", run: "once" do |sh|
    sh.inline = <<~SHELL
    #!/usr/bin/env bash
    wget https://raw.githubusercontent.com/weaveworks-liquidmetal/flintlock/main/hack/scripts/provision.sh
    chmod +x provision.sh
    ./provision.sh devpool
    ./provision.sh containerd --dev
    SHELL
  end

$DOC = <<-'SCRIPT'
  rm -rf ${NETNAME}-net.xml
  cat >> ${NETNAME}-net.xml <<EOF
  <network>
    <name>${NETNAME}</name>
    <forward mode='nat'>
      <nat>
        <port start='1024' end='65535'/>
      </nat>
    </forward>
    <bridge name='${BRIDGE}' stp='on' delay='0'/>
    <ip address='192.168.100.1' netmask='255.255.255.0'>
      <dhcp>
        <range start='192.168.100.10' end='192.168.100.254'/>
      </dhcp>
    </ip>
  </network>
EOF
  virsh net-define ${NETNAME}-net.xml
  virsh net-start ${NETNAME}
  virsh net-autostart ${NETNAME}
  ip tuntap add $TAPNAME mode tap
  ip link set $TAPNAME master $BRIDGE up
  SCRIPT
  
  Vagrant.configure("2") do |config|
    config.vm.provision "shell", inline: $script
  end
  
  config.vm.provision "configure networking", type: "shell", run: "once", privileged: "true" do |sh|
    sh.env = { BRIDGE: "flbr0", TAPNAME: "tap0", NETNAME: "flintlock" }
    sh.inline = $DOC
  end

  config.vm.provision "install-flintlock", type: "shell", run: "once" do |sh|
    sh.inline = <<~SHELL
    #!/usr/bin/env bash
    ./provision.sh firecracker
    ./provision.sh flintlock --dev --insecure --bridge lmbr0 --grpc-address 0.0.0.0:9000
    SHELL
  end

  # install k3s as our mgmt cluster
  config.vm.provision "install-k3s", type: "shell", run: "once" do |sh|
    sh.inline = <<~SHELL
      #!/usr/bin/env bash
      set -eux -o pipefail
      curl -sfL https://get.k3s.io |  sh -s -- --disable=traefik
      cp /etc/rancher/k3s/k3s.yaml /home/vagrant
    SHELL
  end
end