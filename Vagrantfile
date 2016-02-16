VAGRANTFILE_API_VERSION = "2"

base_dir = File.expand_path(File.dirname(__FILE__))
cluster = {
  "mesos-master1" => { :ip => "100.0.10.11",  :cpus => 1, :mem => 1024 },
  "mesos-slave1"  => { :ip => "100.0.10.101", :cpus => 2, :mem => 2048 }
}

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :machine
    config.cache.enable :apt
  end

  config.vm.define "mesos-master1" do |cfg|
    cfg.vm.network "forwarded_port", guest: 5050, host: 5050
    cfg.vm.network "forwarded_port", guest: 2181, host: 2181
    cfg.vm.network "forwarded_port", guest: 8080, host: 8080
  end

  config.vm.define "mesos-slave1" do |cfg|
    cfg.vm.network "forwarded_port", guest: 5051, host: 5051
  end

  cluster.each do |hostname, info|

    config.vm.define hostname do |cfg|

      cfg.vm.provider :virtualbox do |vb, override|
        override.vm.box = "vivid64"
        override.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/vivid/current/vivid-server-cloudimg-amd64-vagrant-disk1.box"
        override.vm.network :private_network, ip: "#{info[:ip]}"
        override.vm.hostname = hostname

        vb.name = 'vagrant-mesos-' + hostname
        vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpus], "--hwvirtex", "on" ]
      end

      # provision nodes with ansible
      cfg.vm.provision :ansible do |ansible|
        ansible.verbose = "v"

        ansible.inventory_path = base_dir + "/inventory/vagrant"
        ansible.playbook = base_dir + "/cluster.yml"
        ansible.extra_vars = {
          vagrant_ip: "#{info[:ip]}"
        }
        ansible.limit = "#{info[:ip]}" # Ansible hosts are identified by ip
      end

    end

  end

end
