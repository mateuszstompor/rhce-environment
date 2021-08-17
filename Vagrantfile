# Configuration of 4 managed nodes + 1 control node

# Password for root user is "vagrant"

# IPs
# controller 192.168.99.100
# managedX 192.168.99.(100 + X)

# Accessing the nodes
# Each node can be accessed by its short name - controller, manged1, manged2, manged3,manged4
# Alternatively fqdn can be used, e. g. controller.example.com, managed1.example.com 

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
NODES_NUMBER = ENV['NODES_NUMBER'] = '4'
ADDITIONAL_DISK_SIZE = 1024 * 5 # 5GiB
ANSIBLE_USER = ENV['ANSIBLE_USER'] = 'ansible'
ANSIBLE_USER_PASSWORD = ENV['ANSIBLE_USER_PASSWORD'] = 'ansible'
BOX = 'bento/centos-8'
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

Vagrant.configure("2") do |config|
  # Create shared directory - project directory is going to be mapped to /vagrant on the client
  config.vm.synced_folder ".", "/vagrant", type: "rsync"
  config.vm.box_check_update = false
  config.vm.provision "shell", inline: <<-INPUT
    echo Install python interpreter which is mandatory in order to use Ansible
    sudo yum module install -y python36
    if ! id #{ ENV['ANSIBLE_USER'] } > /dev/null 2> /dev/null
    then
      sudo useradd #{ ENV['ANSIBLE_USER'] }
    else
      echo User #{ ENV['ANSIBLE_USER'] } has already been configured
    fi
    echo Setting #{ ENV['ANSIBLE_USER'] } password
    echo #{ ENV['ANSIBLE_USER_PASSWORD'] } | passwd --stdin #{ ENV['ANSIBLE_USER'] }
    echo Allow ansible user to execute all commands disregarding host name, do not ask for password
    echo "#{ ENV['ANSIBLE_USER'] } ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/#{ ENV['ANSIBLE_USER'] }
  INPUT

  (1..NODES_NUMBER.to_i).each do |i|
    config.vm.define "managed#{i}" do |node|
      node.vm.box = BOX
      node.vm.hostname = "managed#{i}"
      node.vm.network "private_network", ip: "192.168.99.#{i + 100}"
      disk_file = "./storage/disk#{i}.vdi"
      node.vm.provider "virtualbox" do |v|
        unless File.exist?(disk_file)
            v.customize ['createhd', '--filename', disk_file, '--size', ADDITIONAL_DISK_SIZE]
        end
        v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', disk_file]
      end
      node.vm.provision "shell", inline: <<-INPUT
        echo Add aliases to the rest of hosts
        sudo echo "127.0.0.1 localhost managed#{i} managed#{i}.example.com" > /etc/hosts
        sudo echo "127.0.1.1 managed#{i}" >> /etc/hosts
        for ((i=1; i<=#{ ENV['NODES_NUMBER'] }; i++))
        do
          sudo echo "192.168.99.10$i managed$i managed$i.example.com" >> /etc/hosts
        done
        sudo echo "192.168.99.100 controller controller.example.com" >> /etc/hosts
      INPUT
    end
  end

  config.vm.define "controller" do |controller|
    controller.vm.box = BOX
    controller.vm.hostname = "controller"
    controller.vm.network "private_network", ip: "192.168.99.100"
    controller.vm.provision "shell", inline: <<-INPUT
      sudo echo "127.0.0.1 localhost controller controller.example.com" > /etc/hosts
      sudo echo "127.0.1.1 controller" >> /etc/hosts
      for ((i=1; i<=#{ ENV['NODES_NUMBER'] }; i++))
      do
        sudo echo "192.168.99.10$i managed$i managed$i.example.com" >> /etc/hosts
      done
      sudo yum install -y epel-release --nogpgcheck
      sudo yum install -y vim --nogpgcheck
      echo Use defined user instead of vagrant
      export SWITCH_USER="sudo su - #{ ENV['ANSIBLE_USER'] }"
      export PROFILE_PATH=/home/vagrant/.bash_profile  
      if grep -Fxq "$SWITCH_USER" $PROFILE_PATH
      then
        echo Skipping step of adding user switch - command already there
      else
        sudo echo $SWITCH_USER >> $PROFILE_PATH
      fi
      sudo -u #{ ENV['ANSIBLE_USER'] } /bin/sh << 'USER_INPUT'
        export SSH_PATH=/home/#{ ENV['ANSIBLE_USER'] }/.ssh
        mkdir -pv $SSH_PATH
        sudo yum install -y vim sshpass
        echo Generate public and private key pairs - id_rsa, id_rsa.pub
        ssh-keygen -N "" -f $SSH_PATH/id_rsa
        echo Add public key to all managed servers
        for ((i=1; i<=#{ ENV['NODES_NUMBER'] }; i++))
        do
          sshpass -p "#{ ENV['ANSIBLE_USER_PASSWORD'] }" ssh-copy-id -f -o StrictHostKeyChecking=no -i $SSH_PATH/id_rsa.pub #{ ENV['ANSIBLE_USER'] }@managed$i 2> /dev/null
        done
USER_INPUT
    INPUT
  end
end
