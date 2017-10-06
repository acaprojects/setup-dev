restart_required = false
private_docker_repo = false

header = <<-HEADER
\033[37m
                                         ``
            /dd.                  `:ohmNMNNNMNmhs.              `dd+
           .NMMm`               /hMds/.`     `-/o.              hMMM-
          `mMMMMy             :mMs.                            oMMMMN`
          yMMMMMM/           yMh.                             :MMMMMMh
         /MMMMMMMN.         yMy                              `NMMMMMMMo
        .NMMMMMMMMd        /Mm                               hMMMMMMMMM-
       `mMMMMMMMMMMs       dM+                              oMMMMMMMMMMm`
       yMMMMMMMMMMMM/     `MM.                             :MMMMMMMMMMMMh
      /MMMMMMMMMMMMMN.    `MM.                            `NMMMMMMMMMMMMM+
     -MMMMMMMMMMMMMMMd     NM:                            dMMMMMMMMMMMMMMM-
    `mMNNNNNNNNNNNNNMMs    yMy                           oMMNNNNNNNNNNNNNMm`
    yMy             .NM/   .NM/                         :MN`             yMh
   +Mm`              :MN.   -NMo                       .NM:              `mM+
  -MM-                sMd    `yMm/                     dMo                -MM-
 `mM+                  dMs     -yNNy+-`      .-/s-    sMd                  +Mm`
 ody                   .dd-      `:ohmNMNNNMNmhs+`   -dd.                   ydo
                                        ```
\033[032m

Spinning up a development environment. One moment...
\033[0m
Grab the latest docs from https://developer.acaprojects.com to get started.
HEADER

# Install Required plugins
plugins_required = %w( vagrant-env vagrant-docker-compose vagrant-exec )

plugins_required.each do |plugin_name|
  unless Vagrant.has_plugin? plugin_name
    system("vagrant plugin install #{plugin_name}")
    restart_required = true
    puts "Installing plugin: #{plugin_name}"
  end
end

# If a private Docker image is being used, login
if File.open('.env').grep(/DOCKER_PASSWORD=/).length > 0
  private_docker_repo = true
  unless Vagrant.has_plugin? 'vagrant-docker-login'
    puts "Installing plugin: vagrant-docker-login"
    system("vagrant plugin install vagrant-docker-login")
    restart_required = true
  end
end

# Restart Vagrant if any new plugin or env var is added
exec "vagrant #{ARGV.join' '}" if restart_required

# Print out lovely ASCII art header if we're on the way up
system "echo '#{header}'" if ARGV[0] == 'up'

Vagrant.configure("2") do |config|
  config.vm.define "ACAEngine"

  # Load env vars from .env file for use here
  config.env.enable

  # Load .env into the guest for use by Ansible playbooks
  config.vm.provision :shell, inline: "echo \"set -o allexport; source /vagrant/.env; set +o allexport\" > /etc/profile.d/load_env.sh"

  # Define the base box
  config.vm.box = "bento/ubuntu-17.04"
  config.vm.network "forwarded_port", guest: 8091, host: 8091, auto_correct: true   # Couchbase
  config.vm.network "forwarded_port", guest: 9200, host: 9200, auto_correct: true   # Elasticsearch
  config.vm.network "forwarded_port", guest: 80, host: ENV['WWW_PORT'] #, auto_correct: true   # Web

  # Randomly generate Engine IDs/secrets
  config.vm.provision :ansible_local, playbook: "ansible/secrets.yml", verbose: true

  # nginx.conf doesn't support environment variables, so substitute now
  config.vm.provision :shell, inline: "sed -i -e \"s/\\$WWW_PORT/" + ENV['WWW_PORT'] + "/g\" /vagrant/config/nginx/nginx.conf"

  # Pull down the modues and demo UI repos
  config.vm.provision :shell, inline: "git clone https://github.com/acaprojects/aca-device-modules --depth=1 /vagrant/aca-device-modules || echo Using existing aca-device-modules repo."
  config.vm.provision :shell, inline: "git clone https://github.com/acaprojects/demo-ui --depth=1 /vagrant/demo-ui || echo Using existing demo-ui repo."

  config.vm.provision :docker
  config.vm.provision :docker_login if private_docker_repo
  config.vm.provision :docker_compose, yml: "/vagrant/docker-compose.yaml", run: "always"

  # Init Elasticsearch: Create aca index and upload our couchbase tamplate
  config.vm.provision :ansible_local, playbook: "ansible/elastic.yml", verbose: true

  # Init Couchbase: Create cluster, add this node, create bucket, create XDCR to elasticsearch
  config.vm.provision :ansible_local, playbook: "ansible/couch.yml", verbose: true

  # Init ACAEngine: Generate node ID, Add localhost domain, add backoffice app
  config.vm.provision :ansible_local, playbook: "ansible/engine.yml", verbose: true

  config.vm.post_up_message = "Install complete. Login to http://localhost:#{ENV['WWW_PORT']}/backoffice/ with the below credentials:\nsupport@aca.im\n#{ENV['CB_PASS']}"

  # Provide a neat way to execute tasks on the app server
  config.exec.commands 'bundle', prepend: 'docker exec -i engine'
  config.exec.commands %w[rails rake], prepend: 'docker exec -i engine bundle exec'
end
