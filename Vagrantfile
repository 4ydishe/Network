MACHINES = {
  :inetRouter => {
    :box_name => "centos/7",
    :net => [
      {type: 'private_network', ip: '192.168.255.1', netmask: "255.255.255.252", virtualbox__intnet: "inet-central"},
    ]
  },

  :centralRouter => {
    :box_name => "centos/7",
    :net => [
      {type: 'private_network', ip: '192.168.255.2', netmask: "255.255.255.252", virtualbox__intnet: "inet-central"},
      {type: 'private_network', ip: '192.168.0.1', netmask: "255.255.255.240", virtualbox__intnet: "central-dir"},
      {type: 'private_network', ip: '192.168.0.33', netmask: "255.255.255.240", virtualbox__intnet: "central-hw"},
      {type: 'private_network', ip: '192.168.0.65', netmask: "255.255.255.192", virtualbox__intnet: "central-wifi"},
      {type: 'private_network', ip: '192.168.2.1', netmask: "255.255.255.192", virtualbox__intnet: "office1-central"},
      {type: 'private_network', ip: '192.168.1.1', netmask: "255.255.255.192", virtualbox__intnet: "office2-central"},
    ]
  },

  :office1Router => {
    :box_name => "centos/7",
    :net => [
      {type: 'private_network', ip: '192.168.2.2', netmask: "255.255.255.192", virtualbox__intnet: "office1-central"},
      {type: 'private_network', ip: '192.168.2.65', netmask: "255.255.255.192", virtualbox__intnet: "office1-test"},
      {type: 'private_network', ip: '192.168.2.129', netmask: "255.255.255.192", virtualbox__intnet: "office1-mgr"},
      {type: 'private_network', ip: '192.168.2.193', netmask: "255.255.255.192", virtualbox__intnet: "office1-hw"},
    ]
  },

  :office2Router => {
    :box_name => "centos/7",
    :net => [
      {type: 'private_network', ip: '192.168.1.2', netmask: "255.255.255.192", virtualbox__intnet: "office2-central"},
      {type: 'private_network', ip: '192.168.1.129', netmask: "255.255.255.192", virtualbox__intnet: "office2-test"},
      {type: 'private_network', ip: '192.168.1.193', netmask: "255.255.255.192", virtualbox__intnet: "office2-hw"},
    ]
  },

  :centralServer => {
    :box_name => "centos/7",
    :net => [
      {type: 'private_network', ip: '192.168.0.2', netmask: "255.255.255.240", virtualbox__intnet: "central-dir"},
    ]
  },

  :office1Server => {
    :box_name => "centos/7",
    :net => [
      {type: 'private_network', ip: '192.168.2.3', netmask: "255.255.255.192", virtualbox__intnet: "office1-central"},
    ]
  },

  :office2Server => {
    :box_name => "centos/7",
    :net => [
      {type: 'private_network', ip: '192.168.1.3', netmask: "255.255.255.192", virtualbox__intnet: "office2-central"},
    ]
  },
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s

      boxconfig[:net].each do |netconf|
        box.vm.network netconf[:type], ip: netconf[:ip], netmask: netconf[:netmask], virtualbox__intnet: netconf[:virtualbox__intnet]
      end

      box.vm.provision "shell", inline: <<-SHELL
        sysctl -w net.ipv4.ip_forward=1
        SHELL

      if boxname == :inetRouter
        box.vm.provision "shell", inline: <<-SHELL
          iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
          SHELL
      end
    end
  end
end