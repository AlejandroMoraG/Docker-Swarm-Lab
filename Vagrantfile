# Script to install docker
$install_docker_script = <<SCRIPT
echo Instalando Docker...
curl -sSL https://get.docker.com/ | sh
usermod -aG docker ubuntu
SCRIPT

# Script to config the manager and run de visualizer
$manager_script = <<SCRIPT
echo Swarm Init...
docker swarm init --listen-addr 10.100.99.100:2377 --advertise-addr 10.100.99.100:2377
docker swarm join-token --quiet worker > /vagrant/worker_token
docker run -it -d -p 5000:8080 -v /var/run/docker.sock:/var/run/docker.sock dockersamples/visualizer
SCRIPT

# Script to join the workers with the manager
$worker_script = <<SCRIPT
echo Swarm Join...
docker swarm join --token $(cat /vagrant/worker_token) 10.100.99.100:2377
SCRIPT

Vagrant.configure('2') do |config|

  vm_box = 'ubuntu/bionic64'

  config.vm.define :manager, primary: true  do |manager|
    manager.vm.box = vm_box
    manager.vm.box_check_update = true
    manager.vm.network :private_network, ip: "10.100.99.100"
    manager.vm.network :forwarded_port, guest: 8080, host: 8080
    manager.vm.network :forwarded_port, guest: 5000, host: 5000
    manager.vm.hostname = "manager"
    manager.vm.synced_folder ".", "/vagrant"
    manager.vm.provision "shell", inline: $install_docker_script, privileged: true
    manager.vm.provision "shell", inline: $manager_script, privileged: true
    manager.vm.provider "virtualbox" do |vb|
      vb.name = "manager"
      vb.memory = "1024"
    end
  end

  (1..3).each do |i|
    config.vm.define "worker0#{i}" do |worker|
      worker.vm.box = vm_box
      worker.vm.box_check_update = true
      worker.vm.network :private_network, ip: "10.100.99.10#{i}"
      worker.vm.hostname = "worker0#{i}"
      worker.vm.synced_folder ".", "/vagrant"
      worker.vm.provision "shell", inline: $install_docker_script, privileged: true
      worker.vm.provision "shell", inline: $worker_script, privileged: true
      worker.vm.provider "virtualbox" do |vb|
        vb.name = "worker0#{i}"
        vb.memory = "1024"
      end
    end
  end

end
