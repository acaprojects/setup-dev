# List plugins dependencies
plugins_dependencies = %w( vagrant-docker-compose vagrant-docker-login )
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
  config.vm.box = "ubuntu/xenial64"
  config.vm.provision :docker
  config.vm.provision :docker_compose, yml: "/home/aca-apps/setup-dev/docker-compose.yml", run: "always", compose_version: "2.1"
end
