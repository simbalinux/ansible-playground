# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|
  config.vm.define  "master" do |host|
    host.vm.box = "ubuntu/xenial64"
    host.vm.hostname = "master"
    host.vm.network "private_network", ip: "10.10.10.10"
    host.vm.provision "shell", inline: $script, privileged: false
  end
  config.vm.define  "ubuntu1" do |host|
    host.vm.box = "ubuntu/xenial64"
    host.vm.hostname = "ubuntu1"
    host.vm.network "private_network", ip: "10.10.10.11"
    #host.vm.provision "shell", inline: "sudo apt-get update; sudo ln -sf /usr/bin/python3 /usr/bin/python"     
  end
  config.vm.define  "centos1" do |host|
    host.vm.box = "centos/7"
    host.vm.hostname = "centos1"
    host.vm.network "private_network", ip: "10.10.10.12"
    #host.vm.provision "shell", inline: "sudo yum update -y; sudo ln -sf /usr/bin/python3 /usr/bin/python"     
    host.vm.provision "shell", inline: "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config"     
  end

  config.vm.define  "dnsmasq" do |host|
    host.vm.box = "ubuntu/xenial64"
    host.vm.hostname = "dnsmasq"
    host.vm.network "private_network", ip: "10.10.10.13"
    #host.vm.provision "shell", inline: "sudo apt-get update; sudo ln -sf /usr/bin/python3 /usr/bin/python"     
  end
end
# master provision script
$script = <<-SCRIPT

# if !ansible then install and configure
set -x
if [ ! $(command -v ansible) ]; then 

  # append hosts file
  echo "10.10.10.11      ubuntu1"  | sudo  tee -a /etc/hosts             
  echo "10.10.10.12      centos1"  | sudo  tee -a /etc/hosts             
  echo "10.10.10.13      dnsmasq"  | sudo  tee -a /etc/hosts             
  sleep 5
  # install and configure ansible
  sudo apt-get update 
  mkdir $HOME/ansible && cd $HOME/ansible 
  git clone https://github.com/spurin/masteringansible.git
  sudo apt-get install software-properties-common -y 
  sudo apt-add-repository --yes --update ppa:ansible/ansible 
  sudo apt-get update
  sudo apt-get install ansible -y
  
  # setup direnv  
  sudo apt-get install wget curl jq go tree vim -y
  cd /tmp
  wget https://github.com/direnv/direnv/releases/download/v2.18.2/direnv.linux-amd64
  
  chmod +x direnv.linux-amd64
  
  sudo mv $PWD/direnv.linux-amd64 /usr/local/bin/direnv
  echo 'eval "$(direnv hook bash)"' >> $HOME/.bashrc
  source $HOME/.bashrc
fi

# if KEY !exist create
if [ -f $HOME/.ssh/id_rsa ]; then
  echo rsa key installed
else
  ssh-keygen -f $HOME/.ssh/id_rsa -t rsa -N ''
fi
SCRIPT
