if ARGV[0] == 'up'
  require 'open-uri'
  require 'yaml'

  token = open('https://discovery.etcd.io/new').read

  data = YAML.load(File.read('user-data.yml'))
  data['coreos']['etcd']['discovery'] = token

  File.write('user-data.yml', "#cloud-config\n\n#{YAML.dump(data)}")
end

Vagrant.configure('2') do |config|
  config.ssh.insert_key = false

  config.vm.box = 'coreos-alpha'
  config.vm.box_version = '>= 598.0.0'
  config.vm.box_url = 'http://alpha.release.core-os.net/amd64-usr/current' \
                      '/coreos_production_vagrant.json'

  config.vm.provider :virtualbox do |vb|
    cpus = 1
    mem  = 1024

    if RbConfig::CONFIG['host_os'] =~ /darwin/
      cpus = `sysctl -n hw.ncpu`.to_i
      mem  = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
    elsif RbConfig::CONFIG['host_os'] =~ /linux/
      cpus = `nproc`.to_i
      mem  = `grep 'MemTotal' /proc/meminfo | \
              sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
    end

    vb.check_guest_additions = false
    vb.functional_vboxsf = false
    vb.gui = false
    vb.memory = mem
    vb.cpus = cpus.ceil
  end

  config.vbguest.auto_update = false if Vagrant.has_plugin?('vagrant-vbguest')

  config.vm.define 'blendle-drone' do |machine|
    machine.vm.hostname = 'blendle-drone'
    machine.vm.network :forwarded_port, guest: 80, host: 8080
    machine.vm.network :private_network, type: 'dhcp'

    machine.vm.provision :file,
                         source: './user-data.yml',
                         destination: '/tmp/vagrantfile-user-data'

    machine.vm.provision :shell, privileged: true, inline: <<-EOS
      mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/

      sleep 3
      fleetctl load /etc/systemd/system/drone.service
      fleetctl start drone.service
    EOS
  end
end
