# --- About ---
# Configuration of 4 managed nodes + 1 control node

# --- Good to know ---
# Password for root user is "vagrant"

# --- IPs ---
# controller 192.168.56.100
# managedX 192.168.56.(100 + X)
# managedX 192.168.57.(100 + X)
# Where X ranges from 1 to 4

# --- Accessing the nodes ---
# Each node can be accessed by its short name - controller, manged1, manged2, manged3,manged4
# Alternatively fqdn can be used, e. g. controller.example.com, managed1.example.com

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
NODES_NUMBER = ENV['NODES_NUMBER'] = '4'
ADDITIONAL_DISK_SIZE = 1024 * 5 # 5GiB
USER = ENV['USER'] = 'root'
USER_HOME = ENV['USER_HOME'] = '/root'
USER_PASSWORD = ENV['USER_PASSWORD'] = 'vagrant'
BOX = 'bento/centos-8'
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

Vagrant.configure '2' do |config|
  config.ssh.username = USER
  config.ssh.password = USER_PASSWORD
  config.ssh.insert_key = 'true'
  config.vm.synced_folder ".", "/vagrant", type: "rsync"
  config.vm.box_check_update = false
  config.vm.provision "shell", inline: <<-INPUT
    # # # # # # BEGIN: Install python interpreter mandatory to use Ansible
    sudo sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
    sudo sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

    sudo yum module install -y python36
    # # # # # # END
  INPUT

  (1..NODES_NUMBER.to_i).each do |i|
    config.vm.define "managed#{i}" do |node|
      disk_file = "./storage/disk#{i}.vdi"
      node.vm.box = BOX
      node.vm.hostname = "managed#{i}"
      node.vm.network "private_network", ip: "192.168.56.#{i + 100}"
      node.vm.network "private_network", ip: "192.168.57.#{i + 100}", auto_config: false
      node.vm.provider "virtualbox" do |v|
        unless File.exist? disk_file
            v.customize ['createhd', '--filename', disk_file, '--size', ADDITIONAL_DISK_SIZE]
        end
        v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', disk_file]
      end
      node.vm.provision "shell", inline: <<-INPUT
        # # # # # # BEGIN: Define fqdn and short names
        sudo echo "127.0.0.1 localhost managed#{i} managed#{i}.example.com" > /etc/hosts
        sudo echo "127.0.1.1 managed#{i}" >> /etc/hosts
        for ((i=1; i<=#{ ENV['NODES_NUMBER'] }; i++))
        do
          sudo echo "192.168.56.10$i managed$i managed$i.example.com" >> /etc/hosts
        done
        sudo echo "192.168.56.100 controller controller.example.com" >> /etc/hosts
        # # # # # # END
      INPUT
    end
  end

  config.vm.define "controller" do |controller|
    controller.vm.box = BOX
    controller.vm.hostname = "controller"
    controller.vm.network "private_network", ip: "192.168.56.100"
    controller.vm.provision "shell", inline: <<-INPUT
      # # # # # # BEGIN: Define fqdn and short names
      sudo echo "127.0.0.1 localhost controller controller.example.com" > /etc/hosts
      sudo echo "127.0.1.1 controller" >> /etc/hosts
      for ((i=1; i<=#{ ENV['NODES_NUMBER'] }; i++))
      do
        sudo echo "192.168.56.10$i managed$i managed$i.example.com" >> /etc/hosts
      done
      # # # # # # END

      sudo sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
      sudo sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

      # # # # # # BEGIN: Install man pages
      sudo yum install -y man-pages
      # # # # # # END

      # # # # # # BEGIN: Install editing tools and repo containing ansible
      sudo yum install -y epel-release --nogpgcheck

      sudo sed -i 's|^metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-8&arch=$basearch&infra=$infra&content=$contentdir|baseurl=https://dl.fedoraproject.org/pub/archive/epel/8.5.2022-05-10/Everything/x86_64/|' /etc/yum.repos.d/epel.repo
      sudo yum update --disablerepo="*" --enablerepo="epel" -y

      sudo yum install -y vim sshpass --nogpgcheck
      # # # # # # END

      # # # # # # BEGIN: Define path where ssh keys are going to be stored
      export SSH_PATH=#{ ENV['USER_HOME'] }/.ssh
      # # # # # # END

      # # # # # # BEGIN: Generate public and private key pairs - id_rsa, id_rsa.pub
      mkdir -pv $SSH_PATH
      ssh-keygen -N "" -f $SSH_PATH/id_rsa
      # # # # # # END

      # # # # # # BEGIN: Add fingerprints of all managed servers
      for ((i=1; i<=#{ ENV['NODES_NUMBER'] }; i++))
      do
        for name in {managed$i.example.com,managed$i}
        do
          export FINGERPRINT=$(ssh-keyscan -t rsa $name 2> /dev/null)
          if ! grep -Fxq "$FINGERPRINT" ~/.ssh/known_hosts 2> /dev/null
          then
              echo $FINGERPRINT >> ~/.ssh/known_hosts
          fi
        done
      done
      # # # # # # END

      # # # # # # BEGIN: Push public key to all managed servers
      for ((i=1; i<=#{ ENV['NODES_NUMBER'] }; i++))
      do
        sshpass -p "#{ ENV['USER_PASSWORD'] }" ssh-copy-id -f -i $SSH_PATH/id_rsa.pub #{ ENV['USER'] }@managed$i 2> /dev/null
      done
      # # # # # # END
    INPUT
  end
end
