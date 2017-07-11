# List plugins dependencies
plugins_dependencies = %w( vagrant-env vagrant-docker-compose )
plugin_status = false
plugins_dependencies.each do |plugin_name|
  unless Vagrant.has_plugin? plugin_name
    system("vagrant plugin install #{plugin_name}")
    plugin_status = true
    puts " #{plugin_name}  Dependencies installed"
  end
end
 
# Restart Vagrant if any new plugin installed
if plugin_status === true
  exec "vagrant #{ARGV.join' '}"
else
  puts "All Plugin Dependencies already installed"
end

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.10"

  # Randomly generate passwords and Engine IDs/secrets
  config.vm.provision :ansible_local do |ansible|
    ansible.playbook       = "ansible/secrets.yml"
    ansible.verbose        = true
  end

  config.env.enable  # Load env vars from .env file
  config.vm.network "forwarded_port", guest: 8091, host: 8091, auto_correct: true   # Couchbase
  config.vm.network "forwarded_port", guest: 9200, host: 9200, auto_correct: true   # Elasticsearch
  config.vm.network "forwarded_port", guest: 80, host: ENV['WWW_PORT'] #, auto_correct: true   # Web

  # nginx.conf doesn't support environment variables, so substitute now
  config.vm.provision :shell, inline: "sed -i -e \"s/\\$WWW_PORT/" + ENV['WWW_PORT'] + "/g\" /vagrant/config/nginx/nginx.conf"

  # Link .env file to Vagrant's working directory so docker-compose detects it
  config.vm.provision :shell, inline: "ln -sf /vagrant/.env"
  config.vm.provision :docker
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
    ansible.playbook       = "ansible/couch.yml"
    ansible.verbose        = true
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
end
