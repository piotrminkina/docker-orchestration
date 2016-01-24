require 'yaml'

infrastructure = YAML.load_file('infrastructure.yml')

Vagrant.configure('2') do |config|
  config.vm.box = 'williamyeh/ubuntu-trusty64-docker'
  config.vm.box_check_update = true

  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.ssh.forward_agent = true
  config.ssh.insert_key = false

  infrastructure.each do |groupname, groupconfig|
    parameters = groupconfig['parameters']

    groupconfig['nodes'].each do |nodeconfig|
      config.vm.define nodeconfig['name'] do |node|
        node.vm.network 'private_network',
          ip: nodeconfig['ip'],
          netmask: nodeconfig['netmask']

        config.vm.provider 'virtualbox' do |provider|
          provider.name = "#{nodeconfig['name']}.#{groupname}"
          provider.cpus = parameters['cpus']
          provider.memory = parameters['memory']
          provider.customize ['modifyvm', :id, '--nictype1', 'virtio']
          provider.customize ['modifyvm', :id, '--nictype2', 'virtio']
          provider.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
        end
      end
    end
  end
end
