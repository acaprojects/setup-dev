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
plugins_required = %w[
  vagrant-env
  vagrant-docker-compose
  vagrant-exec
  vagrant-triggers
]

# Ensure the required plugins are available, and restart the process if needed
exec "vagrant #{ARGV.join' '}" if plugins_required.any? do |plugin_name|
  unless Vagrant.has_plugin? plugin_name
    system "vagrant plugin install #{plugin_name}"
  end
end

Vagrant.configure("2") do |config|
  config.vm.define "ACAEngine"

  # Ensure there are no Windows line endings in .env
  config.vm.provision :shell,inline: "sed -i 's/\r$//' /vagrant/.env"

  # Randomly generate Engine IDs/secrets
  config.vm.provision :ansible_local, playbook: "ansible/secrets.yml", verbose: true

  # Load env vars from .env file for use here
  config.env.enable
  # Load .env into the guest for use by Ansible playbooks
  config.vm.provision :shell, inline: 'echo "set -o allexport; source /vagrant/.env; set +o allexport" > /etc/profile.d/load_env.sh'

  # Define the base box
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.network "forwarded_port", guest: 8091, host: 8091, auto_correct: true   # Couchbase
  config.vm.network "forwarded_port", guest: 9200, host: 9200, auto_correct: true   # Elasticsearch
  config.vm.network "forwarded_port", guest: 80, host: ENV['WWW_PORT'] #, auto_correct: true   # Web

  # nginx.conf doesn't support environment variables, so substitute now
  config.vm.provision :shell, inline: "sed -i -e \"s/\\$WWW_PORT/" + ENV['WWW_PORT'] + "/g\" /vagrant/config/nginx/nginx.conf"

  # Pull down the modules and demo UI repos, or update them (if possible)
  aca_repo = <<-SCRIPT
    if test -d $3; then
      echo "Updating $1"
      cd $2
      if ! git pull --no-ff; then
        echo "Could not auto-update at this time"
      fi
    else
      git clone https://github.com/acaprojects/$1 $2 --depth=1 $3
    fi
  SCRIPT
  config.vm.provision :shell, inline: aca_repo, args: ["aca-device-modules", "-b beta", "/vagrant/aca-device-modules"]

  config.vm.provision :docker
  config.vm.provision :docker_compose, yml: "/vagrant/docker-compose.yaml", run: "always"

  # Init Elasticsearch: Create aca index and upload our couchbase tamplate
  config.vm.provision :ansible_local, playbook: "ansible/elastic.yml", verbose: true

  # Init Couchbase: Create cluster, add this node, create bucket, create XDCR to elasticsearch
  config.vm.provision :ansible_local, playbook: "ansible/couch.yml", verbose: true

  # Init ACAEngine: Generate node ID, Add localhost domain, add backoffice app
  config.vm.provision :ansible_local, playbook: "ansible/engine.yml", verbose: true

  # Provide some nice messages as things come up
  config.trigger.before :up do
    puts header
  end
  config.trigger.after :up do
    config.env.load
    puts <<-ACCESS_DETAILS

      Login to http://localhost:#{ENV['WWW_PORT']}/backoffice/ with the credentials below:

          support@aca.im
          #{ENV['CB_PASS']}

    ACCESS_DETAILS
  end

  # Clean up an generated password / applied config
  config.trigger.after :destroy do
    run 'git checkout -- .env config/nginx/nginx.conf'
  end

  # Provide a neat way to execute tasks on the app server
  config.exec.commands 'restart', prepend: 'docker restart' # e.g. 'vagrant exec restart engine', or nginx/couch/elastic
  config.exec.commands 'bundle', prepend: 'docker exec -i engine' # e.g. 'vagrant exec bundle update'
  config.exec.commands %w[rails rake], prepend: 'docker exec -i engine bundle exec'
end
