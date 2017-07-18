# setup-dev

Starter pack for initialising a local, DEV instance of ACA Engine, ideal for Windows/Mac/Linux dev laptops.
Docker containers will run inside an Ubuntu 16 VirtualBox VM.

1. [Install VirtualBox](https://www.virtualbox.org/wiki/Downloads) (works with VMware and Parallels too)
1. [Install Vagrant](https://www.vagrantup.com/docs/installation/)
1. Clone this repo `git clone https://github.com/acaprojects/setup-dev.git`
1. In the setup-dev direcotory, run `vagrant up`
1. Visit `http://localhost:8888/backoffice/`

Don't forget to `vagrant halt` (stop the VM) when not in use.
