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
plugins_required = %w( vagrant-env vagrant-docker-compose )

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

# If a couch password is not already defined
if File.open('.env').grep(/CB_PASS=/).length == 0
  puts "Generating a random 10 character password and appending to .env..."
  open('.env', 'a') do |e|
    e.puts "\nCB_PASS=#{rand(36**10).to_s(36)}"
  end
  restart_required = true
end

# Restart Vagrant if any new plugin or env var is added
exec "vagrant #{ARGV.join' '}" if restart_required

# Print out lovely ASCII art header if we're on the way up
system "echo '#{header}'" if ARGV[0] == 'up'

Vagrant.configure("2") do |config|
  config.vm.define "ACAEngine"

  config.env.enable  # Load env vars from .env file
  config.vm.box =   "bento/ubuntu-16.10"
  config.vm.network "forwarded_port", guest: 8091, host: 8091, auto_correct: true   # Couchbase
  config.vm.network "forwarded_port", guest: 9200, host: 9200, auto_correct: true   # Elasticsearch
  config.vm.network "forwarded_port", guest: 80, host: ENV['WWW_PORT'] #, auto_correct: true   # Web

  # Randomly generate Engine IDs/secrets
  config.vm.provision :ansible_local do |ansible|
    ansible.playbook       = "ansible/secrets.yml"
    ansible.verbose        = true
  end

  # nginx.conf doesn't support environment variables, so substitute now
  config.vm.provision :shell, inline: "sed -i -e \"s/\\$WWW_PORT/" + ENV['WWW_PORT'] + "/g\" /vagrant/config/nginx/nginx.conf"

  # Link .env file to Vagrant's working directory so docker-compose detects it
  config.vm.provision :shell, inline: "ln -sf /vagrant/.env"
  config.vm.provision :shell, inline: "git clone https://github.com/acaprojects/aca-device-modules --depth=1 /vagrant/aca-device-modules || echo Using existing aca-device-modules repo."

  config.vm.provision :docker
  config.vm.provision :docker_login if private_docker_repo
  config.vm.provision :docker_compose, yml: "/vagrant/docker-compose.yaml", run: "always"


  # Init Elasticsearch: Create aca index and upload our couchbase tamplate
  config.vm.provision :ansible_local do |ansible|
    ansible.playbook       = "ansible/elastic.yml"
    ansible.verbose        = true
    ansible.extra_vars  = {
      es_index: ENV['ES_INDEX']
    }
  end

  # Init Couchbase: Create cluster, add this node, create bucket, create XDCR to elasticsearch
  config.vm.provision :ansible_local do |ansible|
    ansible.playbook    = "ansible/couch.yml"
    ansible.verbose     = true
    ansible.extra_vars  = {
      cb_user:   ENV['CB_USER'],
      cb_pass:   ENV['CB_PASS'],
      cb_bucket: ENV['CB_BUCKET'],
      es_index:  ENV['ES_INDEX']
    }
  end

  # Init ACAEngine: Generate node ID, Add localhost domain, add backoffice app
  config.vm.provision :ansible_local do |ansible|
    ansible.playbook       = "ansible/engine.yml"
    ansible.verbose        = true
    ansible.extra_vars  = {
      engine_password:  ENV['CB_PASS'],
      www_port:         ENV['WWW_PORT']
    }
  end

  config.vm.post_up_message = "Install complete. Login to http://localhost:#{ENV['WWW_PORT']}/backoffice/ with the below credentials:\nsupport@aca.im\n#{ENV['CB_PASS']}"
end
