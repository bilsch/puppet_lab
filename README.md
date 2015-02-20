## A little vagrant based vm environemtn for puppet tinkering

This is a little playground I'm using for puppet tinkering/hacking. Feel free to poke around and/or use what I'm doing in here. Please be careful as I'm doing some bad stuff in puppet.conf ( autosign=true )

## Vagrant

Note that the Vagrantfile is doing some Virtualbox specific stuff for networking. I'm also not using real dns - just hacking up a hosts file. If you add hosts after bringing up the master you need to update its /etc/hosts file ( at least ) - may as well just make puppet do that ;)
