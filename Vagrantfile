slaves = 3
local_domain = 'local'

# Build and write a little /etc/hosts file
hosts = ["172.28.128.200 master master.#{local_domain}"]
(5..(slaves+4)).each do |i|
  hosts.push("172.28.128.#{i} slave#{i} slave#{i}.#{local_domain}")
end
File.open("hosts", "w+") do |file|
  file.puts(hosts.join("\n"))
  file.close
end

Vagrant.configure("2") do |config|
  memory = 2048
  autostart = "on" # switch to on to ensure the vm stays up across host reboots
  config.vm.box = "chef/centos-6.5"
  config.vm.provision "shell", inline: "yum -y install http://yum.puppetlabs.com/puppetlabs-release-el-6.noarch.rpm"
  config.vm.provision "shell", inline: "yum -y install centos-release-SCL"
  config.vm.provision "shell", inline: "cat /vagrant/hosts >> /etc/hosts"

  config.vm.define 'master', primary: true do |foo|
    foo.vm.hostname = "master"
    foo.vm.network "private_network", ip: "172.28.128.200", virtualbox__intnet: "puppet"
    foo.vm.network "forwarded_port", guest: 8081, host: 8080
    foo.vm.provision "shell", inline: "yum -y install puppet-server puppet-terminus puppetdb; cp /vagrant/puppet.conf /etc/puppet/puppet.conf; cp /vagrant/puppetdb.conf /etc/puppet/puppetdb.conf"
    foo.vm.provision "shell", inline: "yum -y install ruby193 ; scl enable ruby193 'ruby -v' ; scl enable ruby193 'bash'"
    foo.vm.provision "shell", inline: "gem install librarian-puppet"
    foo.vm.provision "shell", inline: "/sbin/chkconfig puppetdb on ; /sbin/service puppetdb start"
    foo.vm.provision "shell", inline: "/sbin/chkconfig puppetmaster on ; /sbin/service puppetmaster start"
  end

  (5..(slaves+4)).each do |i|
    config.vm.define "slave#{i}" do |foo|
      foo.vm.network "private_network", ip: "172.28.128.#{i}", virtualbox__intnet: "puppet"
      foo.vm.hostname = "slave#{i}"
      foo.vm.provision "shell", inline: "yum -y install puppet centos-release-SCL; cp /vagrant/puppet.conf /etc/puppet/puppet.conf"
      foo.vm.provision "shell", inline: "yum -y install ruby193 ; scl enable ruby193 'ruby -v' ; scl enable ruby193 'bash'"
      foo.vm.provision "shell", inline: "/sbin/chkconfig puppet on ; /sbin/service puppet start"
    end
  end

  config.vm.provider :virtualbox do |vb|
    # Boot with headless mode
    vb.gui = false
    # Use VBoxManage to customize the VM. For example to change memory:
    vb.customize ["modifyvm", :id, "--memory", "#{memory}"]
    vb.customize ["modifyvm", :id, "--autostart-enabled", "#{autostart}"]
  end
end
