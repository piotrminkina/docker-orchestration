require 'yaml'


infrastructure = YAML.load_file('infrastructure.yml')
ansible_groups = {}

Vagrant.configure('2') do |config|
  config.vm.box = 'williamyeh/ubuntu-trusty64-docker'
  config.vm.box_check_update = true

  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.ssh.forward_agent = true
  config.ssh.insert_key = false

  infrastructure['groups'].each do |group_name, group_config|
    machine_resources = group_config['resources']
    ansible_groups[group_name] = []

    group_config['hosts'].each do |host_name, host_config|
      ansible_groups[group_name] << host_name

      config.vm.define host_name, primary: !!host_config['primary'] do |host|
        host.vm.hostname = "#{host_name}"
        host.vm.network 'private_network',
          ip: host_config['ip'],
          netmask: infrastructure['config']['netmask']

        host.vm.provider 'virtualbox' do |vbox|
          vbox.name = "#{host_name}#docker-orchestration"
          vbox.cpus = machine_resources['cpus']
          vbox.memory = machine_resources['memory']
          vbox.customize ['modifyvm', :id, '--nictype1', 'virtio']
          vbox.customize ['modifyvm', :id, '--nictype2', 'virtio']
          vbox.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
        end
      end
    end
  end

  config.vm.define (config.vm.defined_vm_keys.last) do |hosts|
    hosts.vm.provision 'ansible' do |ansible|
      ansible.limit = 'all'
      ansible.playbook = 'provision/site.yml'
      ansible.groups = ansible_groups
    end
  end
end
