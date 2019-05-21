# -*- mode: ruby -*-
# vi: set ft=ruby :
#
# All VMS use linline shell script to enable password auth in ssh & restart the service
Vagrant.configure(2) do |config|
  config.vm.define  "master" do |host|
    host.vm.box = "ubuntu/xenial64"
    host.vm.hostname = "master"
    host.vm.network "private_network", ip: "10.10.10.2"
    host.vm.provision "shell", inline: "sudo apt-get update; sudo ln -sf /usr/bin/python3 /usr/bin/python"     		        # optional if using python3
    # ----- enable for ssh password access -----
    # host.vm.provision "shell", inline: "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd.service"     
    host.vm.provision "shell", inline: $script, privileged: false								# run as non root user && see below for inline script 
  end
  config.vm.define  "ubuntu1" do |host|
    host.vm.box = "ubuntu/xenial64"
    host.vm.hostname = "ubuntu1"
    host.vm.network "private_network", ip: "10.10.10.3"
    host.vm.provision "shell", inline: "sudo apt-get update; sudo ln -sf /usr/bin/python3 /usr/bin/python"     		        # optional if using python3
    # ----- enable for ssh password access -----
    #host.vm.provision "shell", inline: "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd.service"     
    # ----- for pubkey auth -----
    host.vm.provision "shell", inline: "cat /vagrant/pub-keys/id_rsa.pub >> $HOME/.ssh/authorized_keys", privileged: false
  end
  config.vm.define  "centos1" do |host|
    host.vm.box = "centos/7"
    host.vm.hostname = "centos1"
    host.vm.network "private_network", ip: "10.10.10.4"
    # ----- enable for ssh password access -----
    #host.vm.provision "shell", inline: "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd.service"     
    # ----- for pubkey auth -----
    host.vm.provision "shell", inline: "cat /vagrant/pub-keys/id_rsa.pub >> $HOME/.ssh/authorized_keys", privileged: false
  end
  config.vm.define  "dnsmasq" do |host|
    host.vm.box = "ubuntu/xenial64"
    host.vm.hostname = "dnsmasq"
    host.vm.network "private_network", ip: "10.10.10.5"
    host.vm.provision "shell", inline: "sudo apt-get update; sudo ln -sf /usr/bin/python3 /usr/bin/python"     		        # optional if using python3
    # ----- enable for ssh password access -----
    #host.vm.provision "shell", inline: "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd.service"     
    # ----- for pubkey auth -----
    host.vm.provision "shell", inline: "cat /vagrant/pub-keys/id_rsa.pub >> $HOME/.ssh/authorized_keys", privileged: false
  end
end

# inline script for MASTER CENTOS
$script = <<-SCRIPT
# if ! ansible then install and configure
if [ ! $(command -v ansible) ]; then 

  # append hosts file on MASTER NODE
  echo "10.10.10.3      ubuntu1"  | sudo  tee -a /etc/hosts             
  echo "10.10.10.4      centos1"  | sudo  tee -a /etc/hosts             
  echo "10.10.10.5      dnsmasq"  | sudo  tee -a /etc/hosts             
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

# if KEY ! exist create
if [ -f $HOME/.ssh/id_rsa ]; then
  echo rsa key installed
else
# create new key && copy to vagrant folder for other remote hosts
  ssh-keygen -f $HOME/.ssh/id_rsa -t rsa -N ''
  cp $HOME/.ssh/id_rsa.pub /vagrant/pub-keys
fi
# copy pub keys to the remote hosts
#for host in ubuntu1 centos1 dnsmasq1;
#  do 
#  ssh-copy-id -i $HOME/.ssh/id_rsa.pub ${host}
#done
cp -r /vagrant/config/ansible $HOME
SCRIPT



# inline script for MASTER
$script2 = <<-SCRIPT
# if ! ansible then install and configure
if [ ! $(command -v ansible) ]; then 

  # append hosts file on MASTER NODE
  echo "10.10.10.3      ubuntu1"  | sudo  tee -a /etc/hosts             
  echo "10.10.10.4      centos1"  | sudo  tee -a /etc/hosts             
  echo "10.10.10.5      dnsmasq"  | sudo  tee -a /etc/hosts             
  sleep 5
  # install and configure ansible
  sudo yum update -y
  mkdir $HOME/ansible && cd $HOME/ansible 
  git clone https://github.com/spurin/masteringansible.git
  sudo yum install ansible -y
  
  # setup direnv  
  sudo apt-get install wget curl jq go tree vim -y
  cd /tmp
  wget https://github.com/direnv/direnv/releases/download/v2.18.2/direnv.linux-amd64
  
  chmod +x direnv.linux-amd64
  
  sudo mv $PWD/direnv.linux-amd64 /usr/local/bin/direnv
  echo 'eval "$(direnv hook bash)"' >> $HOME/.bashrc
  source $HOME/.bashrc
fi

# if KEY ! exist create
if [ -f $HOME/.ssh/id_rsa ]; then
  echo rsa key installed
else
# create new key && copy to vagrant folder for other remote hosts
  ssh-keygen -f $HOME/.ssh/id_rsa -t rsa -N ''
  cp $HOME/.ssh/id_rsa.pub /vagrant/pub-keys
fi
# copy pub keys to the remote hosts
#for host in ubuntu1 centos1 dnsmasq1;
#  do 
#  ssh-copy-id -i $HOME/.ssh/id_rsa.pub ${host}
#done
cp -r /vagrant/config/ansible $HOME
SCRIPT
