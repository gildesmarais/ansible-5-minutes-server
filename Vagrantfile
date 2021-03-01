# frozen_string_literal: true

VAGRANTFILE_API_VERSION = '2'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.provision 'file', source: '~/.ssh/id_rsa.pub', destination: '~/.ssh/authorized_keys'
  config.vm.provider :virtualbox do |vb|
    vb.customize ['modifyvm', :id, '--memory', '512']
  end

  # Ubuntu 20.04 LTS
  config.vm.define 'ubuntu20' do |ubuntu|
    ubuntu.vm.hostname = 'ubuntu20'
    ubuntu.vm.box = 'ubuntu/focal64'
    ubuntu.vm.network :private_network, ip: '10.0.1.2'
  end
end
