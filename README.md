# ansible-awx

Some notes on building a ansible/awx/kvm/vagrant virtual environment.

## install sshd on guest
```
vagrant winrm -s cmd -c "powershell -Command Get-Service ssh*"
vagrant winrm -e -s cmd -c "powershell -Command Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0"
vagrant winrm -s cmd -c "powershell -Command Set-Service ssh-agent -StartupType Manual"

vagrant winrm -s cmd -c "powershell -Command Start-Service sshd"
vagrant winrm -s cmd -c "powershell -Command Start-Service ssh-agent"

vagrant winrm -e -s cmd -c "ROUTE ADD 0.0.0.0 MASK 0.0.0.0 10.250.90.254 METRIC 50"
```

##  vagrant winrm
`VAGRANT_DISABLE_STRICT_DEPENDENCY_ENFORCEMENT=1 vagrant plugin install winrm winrm-elevated winrm-fs`
`VAGRANT_DISABLE_STRICT_DEPENDENCY_ENFORCEMENT=1 vagrant plugin install vagrant-faster`
`VAGRANT_DISABLE_STRICT_DEPENDENCY_ENFORCEMENT=1 vagrant plugin install vagrant-cachier`


`vagrant winrm -s cmd -c ipconfig`

## virsh / bridge vlan 90 example with vagrant / 2nd NIC
`modprobe 8021q`

cat /etc/sysconfig/network-scripts/ifcfg-enp0s20u3 
```
TYPE=Ethernet
BOOTPROTO=none
DEVICE=enp0s20u3
ONBOOT=yes
BRIDGE=br-enp0s20u3
```

cat /etc/sysconfig/network-scripts/ifcfg-br-enp0s20u3
```
TYPE=Bridge
BOOTPROTO=none
DEVICE=br-enp0s20u3
ONBOOT=yes
DELAY=0
```
```
vconfig add br-enp0s20u3 90
ifup br-enp0s20u3.90
```
cat bridge-vlan90.xml
```
<network>
  <name>vlan90</name>
  <forward mode='bridge'/>
  <bridge name='br-enp0s20u3.90' />
</network>
```

Add vlan bridge to virsh
```
virsh net-list
virsh net-destroy default
virsh net-undefine default

virsh net-define bridge-vlan90.xml
virsh net-start bridge-vlan90.xml
virsh net-autostart bridge-vlan90.xml
```

```
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

Vagrant.configure("2") do |config|
  config.vm.define "cent-01" do |config|
  config.vm.hostname = "cent-01"
  config.vm.box = "centos/7"
  config.vm.box_check_update = false
  config.vm.network :public_network,  :dev => "br-enp0s20u3.90", :mac => "525400aaaaaa"

  config.vm.provision "shell",
    run: "always",
    inline: "sudo ip route add default via 10.250.90.254 dev eth1 metric 50"

  config.vm.provider :libvirt do |v|
    v.memory = 1024
    v.cpu_mode = 'host-passthrough'
    v.qemu_use_session = false
    end
  end
end
```
#end

## sysctl: cannot stat /proc/sys/net/bridge/bridge-nf-call-ip6tables:
nano /etc/sysctl.conf
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
net.bridge.bridge-nf-call-arptables = 0
modprobe bridge
modprobe br_netfilter
sysctl -p /etc/sysctl.conf



