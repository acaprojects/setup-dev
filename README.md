# setup-dev
Starter pack for initialising a local, DEV instance of ACA Engine, ideal for Windows/Mac/Linux dev laptops.
Docker containers will run inside an Ubuntu 16 VirtualBox VM.
For a starter pack running the Docker containers natively (Linux server or laptop), see [setup-prod](https://github.com/acaprojects/setup-prod) repo.

1. [Install VirtualBox](https://www.virtualbox.org/wiki/Downloads)
2. [Install Vagrant](https://www.vagrantup.com/docs/installation/)
3. Edit .env as required
4. In the setup-dev direcotory, run `vagrant up`
5. Visit `http://127.0.0.1:8888/backoffice/`
