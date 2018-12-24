# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "droplet1" do |config|
	  config.vm.provider :digital_ocean do |provider, override|
		  override.ssh.private_key_path = '$$$input_your_private_key_path$$$'
		  override.ssh.username = "vagrant"
		  override.vm.box = 'digital_ocean'
		  override.vm.box_url = "https://github.com/devopsgroup-io/vagrant-digitalocean/raw/master/box/digital_ocean.box"
		  override.nfs.functional = false
		  provider.token = '$$$input_your_token$$$'
		  provider.image = 'ubuntu-16-04-x64'
		  provider.region = 'nyc1'
		  provider.size = '512mb'
	  
	  end
   end
end
