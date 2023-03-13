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

  config.ssh.forward_agent = true
  config.vm.synced_folder "./kc", "/home/vagrant/kc", create: true
# mvm dataplane
  config.vm.network "private_network", type: "dhcp"

  config.vm.network "forwarded_port", guest: 9090, host: 9090
  config.vm.network "forwarded_port", guest: 9001, host: 9001
  config.vm.network "forwarded_port", guest: 6443, host: 6443
  config.vm.network "forwarded_port", guest: 22, host: 2222
  for p in 7443..7473
    config.vm.network "forwarded_port", guest: p, host: p, protocol: "tcp"
  end
  cpus = 2
  memory = 4096
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
  <range start='192.168.100.20' end='192.168.100.200'/>
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
    # sudo ip link set flbr0 master eth0
    # ip tuntap add $TAPNAME mode tap
    # ip link set $TAPNAME master $BRIDGE up
    #   sudo cat >> /etc/sysctl.conf <<EOF
    #   net.bridge.bridge-nf-call-ip6tables = 0
    #   net.bridge.bridge-nf-call-iptables = 0
    #   net.bridge.bridge-nf-call-arptables = 0
    # EOF
    # sudo sysctl -p /etc/sysctl.conf
    # <mac address='08:00:27:ad:8e:67'/>

    config.vm.provision "configure-networking", type: "shell", run: "once", privileged: "true" do |sh|
      sh.env = { BRIDGE: "flbr0", TAPNAME: "tap0", NETNAME: "flintlock" }
      sh.inline = $DOC
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

  config.vm.provision "install-flintlock", type: "shell", run: "once", privileged: "true" do |sh|
    sh.env = { BRIDGE: "flbr0"}
    sh.inline = <<~SHELL
    #!/usr/bin/env bash
    ./provision.sh firecracker -v v1.0.0-macvtap
    ./provision.sh flintlock --dev --insecure --grpc-address 0.0.0.0:9090 --bridge ${BRIDGE}
    SHELL
  end

  # install k3s as our mgmt cluster
  config.vm.provision "install-k3s", type: "shell", run: "once", privileged: "true" do |sh|
    sh.inline = <<~SHELL
      #!/usr/bin/env bash
      set -eux -o pipefail
      curl -sfL https://get.k3s.io  | INSTALL_K3S_EXEC='--flannel-backend=none --disable-network-policy --disable=traefik --node-name capi-mgmt' sh -
      cp /etc/rancher/k3s/k3s.yaml /home/vagrant/kc
      mkdir -p ~/.kube
      cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config
    SHELL
end
  # install cilium as k3s CNI
  config.vm.provision "install-cilium", type: "shell", run: "once" do |sh|
    sh.inline = <<~SHELL
      #!/usr/bin/env bash
      set -eux -o pipefail
      CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/master/stable.txt)
      CLI_ARCH=amd64
      curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
      sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
      sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
      rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
      cilium install --wait
    SHELL
  end
end
