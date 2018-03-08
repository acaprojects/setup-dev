# ACAEngine Development Environment

Starter pack for initialising a local, development instance of ACAEngine, ideal for Windows/Mac/Linux dev laptops. Docker containers will run inside an Ubuntu 17 VirtualBox VM.

1. [Install VirtualBox](https://www.virtualbox.org/wiki/Downloads)
1. [Install Vagrant](https://www.vagrantup.com/docs/installation/)
1. Clone this repo `git clone https://github.com/acaprojects/setup-dev.git`
1. In the setup-dev directory, run `vagrant up`
1. Visit `http://localhost:8888/backoffice/`

Don't forget to `vagrant halt` (stop the VM) when not in use.


## Front-end development

Experiences built on ACAEngine can use any framework (web, or native) that you are comfortable working with by interfacing with our API.

To abstract some of the low-level details though we provide [Composer](https://github.com/acaprojects/ngx-composer), our Angular based application framework. An example application - [ngx-composer-starter](https://github.com/acaprojects/ngx-composer-starter) - pairs with the 'Demo Logic' module loaded as part of this development environment. It will be automatically downloaded when you first boot.


## Further reading

To keep learning more, please head over to the [ACA Developer Guide](https://developer.acaprojects.com/).
