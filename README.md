# setup-dev

Starter pack for initialising a local, DEV instance of ACA Engine, ideal for Windows/Mac/Linux dev laptops.
Docker containers will run inside an Ubuntu 17 VirtualBox VM.

This 'local' branch facilitates deployments with minimal downloading.

### 1) Prep on your machine while you have fast internet: Deploy static files to a portable location
1. [Install Vagrant](https://www.vagrantup.com/docs/installation/)
1. Clone this repo `git clone -b local https://github.com/acaprojects/setup-dev.git`
1. `cd setup-dev`
1. `git clone https://github.com/acaprojects/aca-device-modules.git`
1. `git clone https://github.com/acaprojects/demo-ui.git`
1. Download the base box `vagrant box add acaprojects/engine-base`
1. Compress the base box directory to a file inside setup-dev `tar -czf ~/.vagrant.d/boxes/acaprojects-VAGRANTSLASH-engine-base/0/virtualbox/ ./engine-base.box`

### 2) Deploy on target machine (it still needs internet access)
1. [Install VirtualBox](https://www.virtualbox.org/wiki/Downloads) (works with VMware and Parallels too)
1. [Install Vagrant](https://www.vagrantup.com/docs/installation/)
1. Copy the the setup-dev directory (which should contain aca-device-modules, demo-ui and the engine-base vagrant box) to the target machine then `cd setup-dev`
1. Import the engine-base box into vagrant `vagrant box add acaprojects/engine-base ./engine-base.box`
1. `vagrant up`
1. Vagrant should detect the acaprojects/engine-base, docker, docker-compose, the 4 docker images, and 2 git repos. It will still need to download ansible, but it's pretty small
1. Visit `http://localhost:8888/backoffice/`

Don't forget to `vagrant halt` (stop the VM) when not in use.

## Front end dev uses [demo-ui](https://github.com/acaprojects/demo-ui)
1. `cd demo-ui`
1. `npm install`
1. `gulp serve`
