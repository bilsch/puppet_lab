slaves = 2
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
  autostart = "off" # switch to on to ensure the vm stays up across host reboots
  #config.vm.box = "chef/centos-6.5"
  config.vm.box = "chef/centos-7.0"
  config.vm.provision "shell", inline: "yum -y install http://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm"
  config.vm.provision "shell", inline: "yum -y install http://mirror.oss.ou.edu/epel/7/x86_64/e/epel-release-7-5.noarch.rpm"
  config.vm.provision "shell", inline: "cat /vagrant/hosts >> /etc/hosts"

  config.vm.define 'master', primary: true do |foo|
    foo.vm.hostname = "master.local"
    foo.vm.network "private_network", ip: "172.28.128.200", virtualbox__intnet: "puppet"
    foo.vm.network "forwarded_port", guest: 8081, host: 8080
    foo.vm.provision "shell", inline: "yum -y install rubygem-puppet-lint vim lsof nc puppet-server puppetdb-terminus puppetdb; cp /vagrant/puppet.conf /etc/puppet/puppet.conf; cp /vagrant/puppetdb.conf /etc/puppet/puppetdb.conf;"
    foo.vm.provision "shell", inline: 'yum -y groupinstall "Development tools"'
    foo.vm.provision "shell", inline: 'gem install librarian-puppet'
    foo.vm.provision "shell", inline: "/usr/bin/systemctl enable puppetdb.service ; /usr/bin/systemctl start puppetdb.service"
    foo.vm.provision "shell", inline: "/usr/bin/systemctl enable puppetmaster.service ; /usr/bin/service puppetmaster.service" 
    foo.vm.provision "shell", inline: "puppet agent --test ; puppetdb ssl-setup"
    foo.vm.provider :virtualbox do |vb|
      vb.gui = false
      vb.customize ["modifyvm", :id, "--memory", "2048"]
      vb.customize ["modifyvm", :id, "--autostart-enabled", "#{autostart}"]
    end
  end

  (5..(slaves+4)).each do |i|
    config.vm.define "slave#{i}" do |foo|
      foo.vm.network "private_network", ip: "172.28.128.#{i}", virtualbox__intnet: "puppet"
      foo.vm.hostname = "slave#{i}"
      foo.vm.provision "shell", inline: "yum -y install puppet ; cp /vagrant/puppet.conf /etc/puppet/puppet.conf"
      foo.vm.provision "shell", inline: "/usr/bin/systemctl enable puppet.service ; /usr/bin/systemctl start puppet.service"
      foo.vm.provider :virtualbox do |vb|
        vb.gui = false
        vb.customize ["modifyvm", :id, "--memory", "1024"]
        vb.customize ["modifyvm", :id, "--autostart-enabled", "#{autostart}"]
      end
    end
  end
end
