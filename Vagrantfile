# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.require_plugin "vagrant-libvirt"

$configuration_hash = {
  'HOST_PXE_INTERFACE' => ENV['HOST_PXE_INTERFACE'],
  'FUEL_ADMIN_IP'      => ENV['FUEL_ADMIN_IP'],
  'FUEL_ADMIN_NET_GW'  => ENV['FUEL_ADMIN_NET_GW'],
  'FUEL_ADMIN_MASK'    => ENV['FUEL_ADMIN_MASK'],
  'ADMIN_NET'          => ENV['ADMIN_NET'],
  'DHCP_GW'            => ENV['DHCP_GW'],
  'DHCP_START'         => ENV['DHCP_START'],
  'DHCP_END'           => ENV['DHCP_END'],
  'DNS_SERVER'         => ENV['DNS_SERVER'],
  'NTP_1'              => ENV['NTP_1'],
  'RELEASE_RPM'        => ENV['RELEASE_RPM'],
  'MOS_REPO'           => ENV['MOS_REPO']
}

# Do we have gateway in env, or we want to use provider
# network to access external networks?
$public_network_type = if ENV['FUEL_ADMIN_NET_GW'].to_s.strip.length == 0
                         "nat"
                       else
                         "none"
                       end

Vagrant.configure("2") do |config|
  config.vm.define :fuel_master, primary: true do |fuel|
    fuel.vm.box = "centos/7"
    fuel.vm.box_version = "1610.01"
    fuel.vm.hostname = "fuel"
    if ENV['VIRTUAL'] == 'true'
      fuel.vm.network :private_network,
        ip: ENV['FUEL_ADMIN_IP'],
        libvirt__network_name: "fuel_pxe",
        libvirt__dhcp_enabled: false,
        libvirt__forward_mode: "nat",
        libvirt__host_ip: ENV['FUEL_ADMIN_NET_GW']
    else
      fuel.vm.network :public_network,
        :dev => ENV['HOST_PXE_INTERFACE'],
        :mode => "bridge",
        :type => "bridge",
        :ip => ENV['FUEL_ADMIN_IP']
    end
    fuel.vm.provider :libvirt do |domain|
      domain.memory = 2048
      domain.cpus = 2
      domain.management_network_mode = "#{$public_network_type}"
      domain.storage :file, :size => '50G', :type => 'raw'
    end
    fuel.vm.provision "shell", path: "files/install_fuel.sh", env: $configuration_hash
  end

  (1..ENV['NODES_NUMBER'].to_i).each do |i|
    config.vm.define  "node-#{i}" do |node|
      if ENV['VIRTUAL'] == 'true'
        node.vm.network :private_network,
          libvirt__network_name: "fuel_pxe",
          libvirt__dhcp_enabled: false,
          libvirt__forward_mode: "nat",
          libvirt__host_ip: ENV['FUEL_ADMIN_NET_GW'],
          auto_config: false
        node.vm.network :private_network,
          libvirt__network_name: "fuel_mgmt",
          libvirt__dhcp_enabled: false,
          libvirt__forward_mode: "veryisolated",
          auto_config: false
        node.vm.network :private_network,
          libvirt__network_name: "fuel_storage",
          libvirt__dhcp_enabled: false,
          libvirt__forward_mode: "veryisolated",
          auto_config: false
        node.vm.network :private_network,
          libvirt__network_name: "fuel_private",
          libvirt__dhcp_enabled: false,
          libvirt__forward_mode: "veryisolated",
          auto_config: false
        node.vm.network :private_network,
          libvirt__network_name: "fuel_public",
          libvirt__dhcp_enabled: false,
          libvirt__forward_mode: "nat",
          libvirt__host_ip: ENV['PUBLIC_NET_GW'],
          auto_config: false
      else
        node.vm.network :public_network,
          :dev => ENV['HOST_PXE_INTERFACE'],
          :mode => "bridge",
          :type => "bridge"
        node.vm.network :public_network,
          :dev => ENV['HOST_MGMT_INTERFACE'],
          :mode => "bridge",
          :type => "bridge"
        node.vm.network :public_network,
          :dev => ENV['HOST_STORAGE_INTERFACE'],
          :mode => "bridge",
          :type => "bridge"
        node.vm.network :public_network,
          :dev => ENV['HOST_PRIVATE_INTERFACE'],
          :mode => "bridge",
          :type => "bridge"
        node.vm.network :public_network,
          :dev => ENV['HOST_PUBLIC_INTERFACE'],
          :mode => "bridge",
          :type => "bridge"
      end
      node.vm.provider :libvirt do |domain|
        domain.storage :file, :size => '200G', :type => 'raw'
        domain.memory = 6144
        domain.cpus = 2
        domain.boot 'network'
        domain.nic_model_type = "e1000"
        domain.mgmt_attach = false
      end
    end
  end
  config.vm.synced_folder ".", "/vagrant", disabled: true
end
