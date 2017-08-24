# -*- mode: ruby -*-
# vi: set ft=ruby :

# To build acaprojects/engine-base:
# mv engine-base.Vagrantfile Vagrantfile
# vagrant up
# vagrant vagrant package --base <Virtualbox VM name>
# Upload to https://app.vagrantup.com/acaprojects/boxes/engine-base

# All Vagrant configuration is done below. The "2" in Vagrant.configure
Vagrant.configure("2") do |config|
  config.vm.define "ACAEngine-base"
  config.vm.box = "bento/ubuntu-17.04"
  config.vm.box_check_update = false
  config.ssh.insert_key = false

  config.vm.provision :docker
  config.vm.provision :docker_compose

  config.vm.provision :shell, inline: "docker pull aca0/elastic"
  config.vm.provision :shell, inline: "docker pull couchbase:4.6.2"
  config.vm.provision :shell, inline: "docker pull yobasystems/alpine-nginx:git"
  config.vm.provision :shell, inline: "docker pull aca0/engine:stable"
end
