# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
require 'time'
VAGRANTFILE_API_VERSION = '2'

config_file=File.expand_path(File.join(File.dirname(__FILE__), 'vagrant_variables.yml'))
settings=YAML.load_file(config_file)

LABEL_PREFIX    = settings['label_prefix'] ? settings['label_prefix'] + "-" : ""
NKAFKA           = settings['kafka_vms']
NZOO          = settings['zoo_vms']
PUBLIC_SUBNET   = settings['public_subnet']
CLUSTER_SUBNET  = settings['cluster_subnet']
BOX             = settings['vagrant_box']
BOX_URL         = settings['vagrant_box_url']
SYNC_DIR        = settings['vagrant_sync_dir']
MEMORY          = settings['memory']
ETH             = settings['eth']
USER            = settings['ssh_username']
DEBUG           = settings['debug']

DISABLE_SYNCED_FOLDER = settings.fetch('vagrant_disable_synced_folder', false)
DISK_UUID = Time.now.utc.to_i


ansible_provision = proc do |ansible|
  ansible.playbook = 'setup.yml'

  # Note: Can't do ranges like mon[0-2] in groups because
  # these aren't supported by Vagrant, see
  # https://github.com/mitchellh/vagrant/issues/3539
  ansible.groups = {
    'kafka'             => (0..NKAFKA - 1).map { |j| "#{LABEL_PREFIX}kafka#{j}" },
    'zookeeper'             => (0..NZOO - 1).map { |j| "#{LABEL_PREFIX}zoo#{j}" },
  }

  ansible.extra_vars = {
      cluster_network: "#{CLUSTER_SUBNET}.0/24",
      journal_size: 100,
      #public_network: "#{PUBLIC_SUBNET}.0/24",
  }

# In a production deployment, these should be secret
  ansible.extra_vars = ansible.extra_vars.merge({
    devices: settings['disks'],
  })


  if DEBUG then
    ansible.verbose = '-vvvv'
  end
  ansible.limit = 'all'
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = BOX
  config.vm.box_url = BOX_URL
  config.ssh.insert_key = false # workaround for https://github.com/mitchellh/vagrant/issues/5048
  config.ssh.private_key_path = settings['ssh_private_key_path']
  config.ssh.username = USER

  # Faster bootup. Disables mounting the sync folder for libvirt and virtualbox
  if DISABLE_SYNCED_FOLDER
    config.vm.provider :virtualbox do |v,override|
      override.vm.synced_folder '.', SYNC_DIR, disabled: true
    end
  end

  (0..NKAFKA - 1).each do |i|
    config.registration.skip = true
    config.vm.define "#{LABEL_PREFIX}kafka#{i}" do |awx|
      #osd.vm.network :private_network,
      #  ip: "#{PUBLIC_SUBNET}.10#{i}"
      awx.vm.network :private_network,
        ip: "#{CLUSTER_SUBNET}.10#{i}",
        virtualbox__intnet: true
      awx.vm.hostname = "#{LABEL_PREFIX}kafka#{i}"
      # Virtualbox
      awx.vm.provider :virtualbox do |vb|
        vb.customize ['modifyvm', :id, '--memory', "#{MEMORY}"]
      end
    end
  end
 
  (0..NZOO - 1).each do |i|
    config.registration.skip = true
    config.vm.define "#{LABEL_PREFIX}zoo#{i}" do |db|
      db.vm.network :private_network,
        ip: "#{CLUSTER_SUBNET}.20#{i}",
        virtualbox__intnet: true
      db.vm.hostname = "#{LABEL_PREFIX}zoo#{i}"
      # Virtualbox
      db.vm.provider :virtualbox do |vb|
        vb.customize ['modifyvm', :id, '--memory', "#{MEMORY}"]
      end
    end
  end
end
