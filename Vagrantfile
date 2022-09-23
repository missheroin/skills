INTENT_TYPE="internal-net"

MACHINES = {
  :ISP => {
             :box_name => "centos7",
             :net => [
                      {ip: '200.100.100.1', adapter: 2, netmask: "255.255.255.0",virtualbox__intnet: "CLI2ISP"},
                      {ip: '20.20.20.1', adapter: 3, netmask: "255.255.255.0",virtualbox__intnet: "BR2ISP"},
                      {ip: '10.10.10.1', adapter: 4, netmask: "255.255.255.252", virtualbox__intnet: "HQ2ISP"},
                     ],
            },
##HQ            
  :HQLinRTR => {
             :box_name => "centos7",
             :net => [
                      {ip: '192.168.0.1', adapter: 2, netmask: "255.255.255.0",virtualbox__intnet: "HQ2SRV"},
                      {ip: '10.10.10.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "HQ2ISP"},
                     ],
            },
  :HQLinSRV1 => {
             :box_name => "centos7",
             :net => [
                      {ip: '192.168.0.10', adapter: 2, netmask: "255.255.255.0",virtualbox__intnet: "HQ2SRV"},
                     ],
            },
  :HQLinSRV2 => {
             :box_name => "centos7",
             :net => [
                      {ip: '192.168.0.20', adapter: 2, netmask: "255.255.255.0",virtualbox__intnet: "HQ2SRV"},
                     ],
            },
  :HQLinSRV3 => {
             :box_name => "centos7",
             :net => [
                      {ip: '192.168.0.30', adapter: 2, netmask: "255.255.255.0",virtualbox__intnet: "HQ2SRV"},
                     ],
            },
  :HQCLI => {
             :box_name => "centos7",
             :net => [
#                      {ip: '192.168.0.30', adapter: 2, netmask: "255.255.255.0",virtualbox__intnet: "HQ2SRV"},
                      {type: "dhcp",virtualbox__intnet: "HQ2SRV"},
                     ],
            },
##BR
  :BRLinRTR => {
             :box_name => "centos7",
             :net => [
                      {ip: '20.20.20.2', adapter: 2, netmask: "255.255.255.0",virtualbox__intnet: "BR2ISP"},
                      {ip: '172.16.0.1', adapter: 3, netmask: "255.255.255.0",virtualbox__intnet: "BR2SRV"},
                     ],
            },
  :BRLinSRV => {
             :box_name => "centos7",
             :net => [
                      {ip: '172.16.0.10', adapter: 2, netmask: "255.255.255.0",virtualbox__intnet: "BR2SRV"},
                     ],
            },
  :BRLinCLI => {
             :box_name => "centos7",
             :net => [
                      {ip: '172.16.0.100', adapter: 2, netmask: "255.255.255.0",virtualbox__intnet: "BR2SRV"},
                     ],
            },
##RemoteCLI
  :RemoteLinCLI => {
             :box_name => "centos7",
             :net => [
                      {ip: '200.100.100.100', adapter: 2, netmask: "255.255.255.0",virtualbox__intnet: "CLI2ISP"},
                     ],
            },
}

MACHINES.each do |hostname,config|  
  config[:net].each do |ip|
    if ip[:virtualbox__intnet]==INTENT_TYPE
      hosts_file=hosts_file+ip[:ip]+"\t"+hostname.to_s+"\n"
    end
  end
end

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        box.vm.provider "virtualbox" do |v|
          v.memory = 256
        end
        case boxname.to_s
        when "ISP"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo 'net.ipv4.conf.all.forwarding=1'>>/etc/sysctl.conf
            sysctl net.ipv4.conf.all.forwarding=1
            iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o enp0s3 -j MASQUERADE
			iptables -t nat -A POSTROUTING -s 172.16.0.0/24 -o enp0s3 -j MASQUERADE
			iptables -t nat -A POSTROUTING -s 10.10.10.0/30 -o enp0s3 -j MASQUERADE
			iptables -t nat -A POSTROUTING -s 20.20.20.0/24 -o enp0s3 -j MASQUERADE
			iptables -t nat -A POSTROUTING -s 200.100.100.0/24 -o enp0s3 -j MASQUERADE
			ip route add 192.168.0.0/24 via 10.10.10.2 dev enp0s10 # HQ
			ip route add 172.16.0.0/24 via 20.20.20.2 dev enp0s9 # BR
			echo '#!/bin/bash'>>/etc/init.d/restart.sh
			echo 'ip route add 192.168.0.0/24 via 10.10.10.2 dev enp0s10'>>/etc/init.d/restart.sh
			echo 'ip route add 172.16.0.0/24 via 20.20.20.2 dev enp0s9'>>/etc/init.d/restart.sh
			chmod +x /etc/init.d/restart.sh
			chmod +x /etc/rc.local
			systemctl enable rc-local
			echo 'sh /etc/init.d/restart.sh'>>/etc/rc.local
            SHELL
            
        when "HQLinRTR"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
        	echo 'net.ipv4.conf.all.forwarding=1'>>/etc/sysctl.conf
			sysctl net.ipv4.conf.all.forwarding=1
			ip route del default via 10.0.2.2 dev enp0s3
			ip route add default via 10.10.10.1 dev enp0s9
			echo '#!/bin/bash'>>/etc/init.d/restart.sh
			echo 'ip route del default via 10.0.2.2 dev enp0s3'>>/etc/init.d/restart.sh
			echo 'ip route add default via 10.10.10.1 dev enp0s9'>>/etc/init.d/restart.sh
			chmod +x /etc/init.d/restart.sh
			chmod +x /etc/rc.local
			systemctl enable rc-local
			echo 'sh /etc/init.d/restart.sh'>>/etc/rc.local
            SHELL
        when "BRLinRTR"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
			echo 'net.ipv4.conf.all.forwarding=1'>>/etc/sysctl.conf
			sysctl net.ipv4.conf.all.forwarding=1
			ip route del default via 10.0.2.2 dev enp0s3
			ip route add default via 20.20.20.1 dev enp0s8
			echo '#!/bin/bash'>>/etc/init.d/restart.sh
			echo 'ip route del default via 10.0.2.2 dev enp0s3'>>/etc/init.d/restart.sh
			echo 'ip route add default via 20.20.20.1 dev enp0s8'>>/etc/init.d/restart.sh
			chmod +x /etc/init.d/restart.sh
			chmod +x /etc/rc.local
			systemctl enable rc-local
			echo 'sh /etc/init.d/restart.sh'>>/etc/rc.local
            SHELL
        when "HQLinSRV1"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
			ip route del default via 10.0.2.2 dev enp0s3
			ip route add default via 192.168.0.1 dev enp0s8
			echo '#!/bin/bash'>>/etc/init.d/restart.sh
			echo 'ip route del default via 10.0.2.2 dev enp0s3'>>/etc/init.d/restart.sh
			echo 'ip route add default via 192.168.0.1 dev enp0s8'>>/etc/init.d/restart.sh
			chmod +x /etc/init.d/restart.sh
			chmod +x /etc/rc.local
			systemctl enable rc-local
			echo 'sh /etc/init.d/restart.sh'>>/etc/rc.local
			SHELL
        when "HQLinSRV2"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
			ip route del default via 10.0.2.2 dev enp0s3
			ip route add default via 192.168.0.1 dev enp0s8
			echo '#!/bin/bash'>>/etc/init.d/restart.sh
			echo 'ip route del default via 10.0.2.2 dev enp0s3'>>/etc/init.d/restart.sh
			echo 'ip route add default via 192.168.0.1 dev enp0s8'>>/etc/init.d/restart.sh
			chmod +x /etc/init.d/restart.sh
			chmod +x /etc/rc.local
			systemctl enable rc-local
			echo 'sh /etc/init.d/restart.sh'>>/etc/rc.local
            SHELL
        when "HQLinSRV3"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
			ip route del default via 10.0.2.2 dev enp0s3
			ip route add default via 192.168.0.1 dev enp0s8
			echo '#!/bin/bash'>>/etc/init.d/restart.sh
			echo 'ip route del default via 10.0.2.2 dev enp0s3'>>/etc/init.d/restart.sh
			echo 'ip route add default via 192.168.0.1 dev enp0s8'>>/etc/init.d/restart.sh
			chmod +x /etc/init.d/restart.sh
			chmod +x /etc/rc.local
			systemctl enable rc-local
			echo 'sh /etc/init.d/restart.sh'>>/etc/rc.local
			SHELL
        when "HQCLI"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
			ip route del default via 10.0.2.2 dev enp0s3
			ip route add default via 192.168.0.1 dev enp0s8
			echo '#!/bin/bash'>>/etc/init.d/restart.sh
			echo 'ip route del default via 10.0.2.2 dev enp0s3'>>/etc/init.d/restart.sh
			echo 'ip route add default via 192.168.0.1 dev enp0s8'>>/etc/init.d/restart.sh
			chmod +x /etc/init.d/restart.sh
			chmod +x /etc/rc.local
			systemctl enable rc-local
			echo 'sh /etc/init.d/restart.sh'>>/etc/rc.local
            SHELL
        when "BRLinSRV"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
			ip route del default via 10.0.2.2 dev enp0s3
			ip route add default via 172.16.0.1 dev enp0s8
			echo '#!/bin/bash'>>/etc/init.d/restart.sh
			echo 'ip route del default via 10.0.2.2 dev enp0s3'>>/etc/init.d/restart.sh
			echo 'ip route add default via 172.16.0.1 dev enp0s8'>>/etc/init.d/restart.sh
			chmod +x /etc/init.d/restart.sh
			chmod +x /etc/rc.local
			systemctl enable rc-local
			echo 'sh /etc/init.d/restart.sh'>>/etc/rc.local
            SHELL
            box.vm.disk :disk, size: "1GB", name: "BRLinSRV_disk1"
            box.vm.disk :disk, size: "1GB", name: "BRLinSRV_disk2"

        when "BRLinCLI"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
			ip route del default via 10.0.2.2 dev enp0s3
			ip route add default via 172.16.0.1 dev enp0s8
			echo '#!/bin/bash'>>/etc/init.d/restart.sh
			echo 'ip route del default via 10.0.2.2 dev enp0s3'>>/etc/init.d/restart.sh
			echo 'ip route add default via 172.16.0.1 dev enp0s8'>>/etc/init.d/restart.sh
			chmod +x /etc/init.d/restart.sh
			chmod +x /etc/rc.local
			systemctl enable rc-local
			echo 'sh /etc/init.d/restart.sh'>>/etc/rc.local
            SHELL
        when "RemoteLinCLI"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
			ip route del default via 10.0.2.2 dev enp0s3
			ip route add default via 200.100.100.1 dev enp0s8
			echo '#!/bin/bash'>>/etc/init.d/restart.sh
			echo 'ip route del default via 10.0.2.2 dev enp0s3'>>/etc/init.d/restart.sh
			echo 'ip route add default via 200.100.100.1 dev enp0s8'>>/etc/init.d/restart.sh
			chmod +x /etc/init.d/restart.sh
			chmod +x /etc/rc.local
			systemctl enable rc-local
			echo 'sh /etc/init.d/restart.sh'>>/etc/rc.local
            SHELL
        end
    end
    config.vm.provision "shell" do |s|
       ssh_pub_key = File.readlines("/home/sirius/.ssh/id_rsa.pub").first.strip
       s.inline = <<-SHELL
         mkdir -p /home/vagrant/.ssh
         sudo mkdir -p /root/.ssh
         echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
         echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
       SHELL
    end
  end
end

