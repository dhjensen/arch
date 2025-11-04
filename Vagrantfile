VAGRANT_API_VERSION = "2"

Vagrant.configure(VAGRANT_API_VERSION) do |config|
  config.vm.box = "bartekn337/TrueNAS-SCALE-22.12.3.3"
  config.vm.synced_folder ".", "/vagrant", disabled: true

  NODE_COUNT = 4
  BASE_NAME  = "vagrant-"
  MEMORY_MB  = 4096
  CPUS       = 2
  DATA_DISK  = "10G"

  (1..NODE_COUNT).each do |i|
    node_name = format("%s%02d", BASE_NAME, i)

    config.vm.define node_name, autostart: (i == 1) do |node|
      node.vm.hostname = node_name

      node.vm.network "public_network", :dev => "wlp0s20f3",
        libvirt__network_name: "default",
        libvirt__dhcp_enabled: true,
        libvirt__type: "bridge",
        libvirt__mode: "bridge",
        auto_config: true

      node.vm.network "private_network",
        ip: "10.30.0.1#{i}",
        libvirt__network_name: "vm_migration_net",
        libvirt__dhcp_enabled: false,
        auto_config: true

      node.vm.network "private_network",
        ip: "10.20.0.1#{i}",
        libvirt__network_name: "cluster_io_net",
        libvirt__dhcp_enabled: false,
        auto_config: true

      node.vm.provider :libvirt do |libvirt|
        libvirt.driver = "kvm"
        libvirt.memory = MEMORY_MB
        libvirt.cpus = CPUS
        libvirt.disk_bus = "virtio"
        libvirt.machine_type = "q35"
        libvirt.nested = true

        if i <= 2
          ["disk1", "disk2"].each do |disk|
            disk_path = File.expand_path("./#{node_name}-#{disk}.qcow2")
            
            libvirt.storage :file,
                              size: "10G",
                              type: "qcow2",
                              file: disk_path,
                              bus: "virtio",
                              allow_existing: true
          end
        end
      end

      node.vm.provision "ansible" do |ansible|
        ansible.playbook = "site.yml"
        ansible.limit = node_name
        ansible.extra_vars = {
          hostname: node_name,
          mgmt_ip: "10.30.0.#{i}",
          cluster_ip: "10.20.0.#{i}"
        }
      end
    end
  end
end
