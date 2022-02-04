# frozen_string_literal: true

# -*- mode: ruby -*-
# vi: set ft=ruby :

# We need Vagrant >= 2.1.0 becuse we use triggers
Vagrant.require_version '>= 2.1.0'

required_plugins = %w[vagrant-reload vagrant-persistent-storage vagrant-vbguest nugrant]
plugins_to_install = required_plugins.reject { |plugin| Vagrant.has_plugin? plugin }
unless plugins_to_install.empty?
  puts "Installing plugins: #{plugins_to_install.join(' ')}"
  if system "vagrant plugin install #{plugins_to_install.join(' ')}"
    exec "vagrant #{ARGV.join(' ')}"
  else
    abort 'Installation of one or more plugins has failed. Aborting.'
  end
end

vagrant_dir = __dir__

# All Vagrant configuration is done below. The '2' in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  # Important: use Bento boxes https://app.vagrantup.com/bento not the Canonical ones.
  # Bento boxes are officially-recommended by Vagrant https://www.vagrantup.com/docs/boxes.html
  config.vm.box = 'bento/ubuntu-20.04'

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Specifying an unsupported audio setting will cause VirtualBox provisioning
  # to fail.
  default_vb_audio = nil
  default_vb_audiocontroler = 'hda'
  if Vagrant::Util::Platform.windows?
    default_vb_audio = 'dsound'
  elsif Vagrant::Util::Platform.platform =~ /darwin/
    default_vb_audio = 'coreaudio'
    default_vb_audiocontroler = 'hda'
  end

  # Customizable configuration
  # See https://github.com/maoueh/nugrant
  config.user.defaults = {    
    'ansible' => {
      'skip_tags' => []
    },

    'virtualbox' => {
      'name' => 'development-environment',
      'gui' => true,
      'cpus' => 2,
      'graphicscontroller' => nil,
      'vram' => '64',
      'accelerate3d' => 'off',
      'memory' => '4096',
      'clipboard' => 'bidirectional',
      'draganddrop' => 'bidirectional',
      'audio' => default_vb_audio,
      'audiocontroller' => default_vb_audiocontroler
    },

    'timezone' => 'Europe/London',

    'locales' => {
      'default' => 'en_GB.UTF-8',
      'present' => ['en_GB.UTF-8', 'en_US.UTF-8']
    },

    'keyboard' => {
      'model' => 'pc105',
      'layout' => 'gb',
      'variant' => ''
    },

    'dock_position' => 'LEFT',

    'git_user' => {
      'name' => nil,
      'email' => nil,
      'force' => false
    },

    # Deprecated use intellij.edition instead
    'intellij_edition' => nil,

    'intellij' => {
      'edition' => 'community',
      'license_key_path' => nil
    },

    'persistent_storage_location' => '.vagrant/persistent-disk.vdi'
  }

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing 'localhost:8080' will access port 80 on the guest machine.
  # config.vm.network 'forwarded_port', guest: 80, host: 8080
  config.vm.network 'forwarded_port', guest: 8080, host: 8080, auto_correct: true, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network 'private_network', ip: '192.168.33.10'

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network 'public_network'

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder '../data', '/vagrant_data'

  config.persistent_storage.enabled = true
  config.persistent_storage.location = config.user.persistent_storage_location
  config.persistent_storage.size = 16_000
  config.persistent_storage.mountname = 'persistent'
  config.persistent_storage.filesystem = 'ext4'
  config.persistent_storage.mountpoint = '/var/persistent'
  config.persistent_storage.volgroupname = 'persist-vg'

  # Update the VirtualBox Guest Additions
  config.vbguest.auto_update = false

  config.vm.provider 'virtualbox' do |vb|
    # Give the VM a name
    vb.name = config.user.virtualbox.name

    # Display the VirtualBox GUI when booting the machine
    vb.gui = config.user.virtualbox.gui

    # Customize CPU settings
    vb.cpus = config.user.virtualbox.cpus

    # Customize graphics settings
    unless config.user.virtualbox.graphicscontroller.nil?
      vb.customize ['modifyvm', :id, '--graphicscontroller', config.user.virtualbox.graphicscontroller]
    end
    vb.customize ['modifyvm', :id, '--vram', config.user.virtualbox.vram]
    vb.customize ['modifyvm', :id, '--accelerate3d', config.user.virtualbox.accelerate3d]

    # Customize the amount of memory on the VM
    vb.memory = config.user.virtualbox.memory

    # Enable host desktop integration
    vb.customize ['modifyvm', :id, '--clipboard', config.user.virtualbox.clipboard]
    vb.customize ['modifyvm', :id, '--draganddrop', config.user.virtualbox.draganddrop]

    # vb.customize ['modifyvm', :id, '--natdnshostresolver1', "on"]

    unless config.user.virtualbox.audio.nil?
      # Enable sound
      vb.customize ['modifyvm', :id, '--audio', config.user.virtualbox.audio, '--audiocontroller', config.user.virtualbox.audiocontroller]
    end
  end

  # Customfile
  #
  # Use this to insert your own (and possibly rewrite) Vagrant config lines.
  # If a file 'Customfile' exists in the same directory as this Vagrantfile,
  # it will be evaluated as ruby inline as it loads.
  if File.exist?(File.join(vagrant_dir, 'Customfile'))
    eval(IO.read(File.join(vagrant_dir, 'Customfile')), binding)
  end

  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define 'atlas' do |push|
  #   push.app = 'YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME'
  # end

  # Use persistent APT cache
  config.vm.provision 'shell', inline: <<SCRIPT
  persistent_mount='/var/persistent/var/cache/apt/archives /var/cache/apt/archives none bind 0 0'
  mkdir -p /var/persistent/var/cache/apt/archives \
  && grep -q -F "${persistent_mount}" /etc/fstab || echo "${persistent_mount}" >> /etc/fstab \
  && mount /var/cache/apt/archives

  persistent_mount='/var/persistent/usr/local/src/ansible/data /usr/local/src/ansible/data none bind 0 0'
  mkdir -p /var/persistent/usr/local/src/ansible/data \
  mkdir -p /usr/local/src/ansible/data \
  && grep -q -F "${persistent_mount}" /etc/fstab || echo "${persistent_mount}" >> /etc/fstab \
  && mount /usr/local/src/ansible/data
SCRIPT

  # Perform preliminary setup before the main Ansible provisioning
  config.vm.provision 'ansible_local' do |ansible|
    ansible.extra_vars = { ansible_python_interpreter: '/usr/bin/python3' }
    ansible.playbook = 'provisioning/init.yml'
    ansible.skip_tags = config.user.ansible.skip_tags
  end

  # Run Ansible from the Vagrant VM
  config.vm.provision 'ansible_local' do |ansible|
    ansible.playbook = 'provisioning/playbook.yml'
    ansible.galaxy_role_file = 'provisioning/requirements.yml'
    ansible.galaxy_roles_path = '/home/vagrant/.ansible/roles'

    ansible.extra_vars = {
      ansible_python_interpreter: '/usr/bin/python3',
      timezone: config.user.timezone,

      locales_present: config.user.locales.present,
      locales_default: {
        lang: config.user.locales['default']
      },
      keyboard_model: config.user.keyboard.model,
      keyboard_layout: config.user.keyboard.layout,
      keyboard_variant: config.user.keyboard.variant,

      xdesktop_dock_position: config.user.dock_position,      

      git_user_name: config.user.git_user.name,
      git_user_email: config.user.git_user.email,
      git_user_force: config.user.git_user.force,

      intellij_edition: config.user.intellij_edition.nil? ? config.user.intellij.edition : config.user.intellij_edition,

      intellij_license_key_path: config.user.intellij.license_key_path
    }
    ansible.skip_tags = config.user.ansible.skip_tags
  end

  # Restart the VM after everything is installed
  config.vm.provision :reload

  # Force password change on first use
  # config.vm.provision 'shell', inline: 'chage --lastday 0 vagrant'
end
