# -*- mode: ruby -*-
# vi: set ft=ruby :
#
$ansible_mode = 'ansible' or 'ansible_local'

VAGRANT_ROOT = File.dirname(File.expand_path(__FILE__))
VAGRANTFILE_API_VERSION = "2"

VIRTUAL_MACHINES = {
  :left => {
    :ip             => '192.168.49.41',
  },

}

def connect_storage(vb, machine_id, label, vdi_filename, port_id,
                    size=16*1024)  # 4GB default
    unless File.exist? vdi_filename
        vb.customize [
                'storagectl', :id, '--name', 'SATA',
                '--add', 'SATA'
        ]
        vb.customize [
                "createhd", "--filename", vdi_filename,
                '--format', 'VDI', '--size', size,
        ]
    end

    vb.customize  [
                    "storageattach", :id,
                    '--storagectl', "SATA",
                    '--port', port_id, '--device', 0, '--type', 'hdd',
                    '--medium', vdi_filename
                    ]

    vb.customize  [
                    "setextradata", :id,
                    "VBoxInternal/Devices/ahci/0/Config/Port#{port_id}/SerialNumber",
                    label]
    vb.customize  [
                    "setextradata", :id,
                    "VBoxInternal/Devices/ahci/0/Config/Port#{port_id}/ModelNumber",
                    "VIRTUAL"]
end


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.hostmanager.enabled = true
  config.vm.box = "centos/7"
  config.vm.box_check_update = false

  config.ssh.insert_key = false

  VIRTUAL_MACHINES.each do |name,cfg|

    config.vm.define name do |vm_config|
      vm_config.vm.hostname = name
      vm_config.vm.network :private_network, ip: cfg[:ip]
      vm_config.vm.synced_folder ".", "/vagrant", type: "nfs"
      # check https://www.vagrantup.com/docs/synced-folders/nfs.html

      config.vm.provider :virtualbox do |vb|
        vb.memory = 2048
        vb.cpus = 2
        vb.linked_clone = true

        docker_vdi_file = VAGRANT_ROOT + "/.vagrant/docker.#{name}.vdi"
        connect_storage(vb, :id, 'docker', docker_vdi_file, 1)

      end # provider

      config.vm.provision $ansible_mode do |ansible|
            ansible.playbook = "site.yml"
            ansible.sudo_user = 'root'
            ansible.sudo = true
            ansible.verbose = "v"

            if ENV['ANSIBLE_TAGS'] then ansible.tags = ENV['ANSIBLE_TAGS']; end

            ansible.groups = {
              'vagrant'       => VIRTUAL_MACHINES.keys, # all nodes
              'docker-nodes'  => VIRTUAL_MACHINES.keys,
           }
            ansible.limit = 'all'
      end

      if Vagrant.has_plugin?("vagrant-cachier")
          config.cache.synced_folder_opts = {
                type: :nfs,
          }
          config.cache.scope = :machine
      end
    end
  end
end

