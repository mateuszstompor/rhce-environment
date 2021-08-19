# RHCE environment
This repository contains a vagrant configuration that describes the infrastructure of nodes that can be used for RHCE exam preparation (EX294). The idea of creating such a similar project in comparison to what's currently available is to provide a set of exercises and solutions that are relevant to the configuration and allow the people preparing for the exam to check the exact steps they need to follow in order to solve them

## Exercises 
The whole purpose of using this project is to practice. Don't waste time, spin up machines and check out [this repository](https://github.com/mateuszstompor/rhce-ex294-exam). More content is going to be added soon 

## Assumptions
Vagrant file has a description of a minimal installation and infrastructure description

### Cluster
There are five nodes in total deployed
* 1 control node
* 4 managed nodes

### User management
Each managed host can be accessed directly through Vagrant/VirtualBox using `root` account. **Root password is `vagrant`**. On top of that managed hosts are accessible from the controller node with use of ssh


### Network
All machines have two network adapters configured. One is used for NAT access and the second one allows hosts to talk to each other

#### Aliases
There are symbolic names associated with each host, here is how the definitions look like
```
192.168.99.100    controller  controller.example.com
192.168.99.101    managed1    managed1.example.com
192.168.99.102    managed2    managed2.example.com
192.168.99.103    managed3    managed3.example.com
192.168.99.104    managed4    managed4.example.com
```

### Storage
Each managed host has two disks bound to it. One is taken by the system and the second one is spare and meant to be used for educational purposes. The controller node has a single drive attached

## How to install
### macOS
1. Install homebrew
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
2. Install VirtualBox
```
brew install --cask virtualbox
```

3. Install Vagrant
```
brew install --cask vagrant
```

4. Check out the repo
```
git clone https://github.com/mateuszstompor/rhce-environment.git
```

5. Navigate to the repo
```
cd rhce-environment
```

6. Provision
```
vagrant up --provision
``` 

## Commonly used commands
### Accessing the machines
In order to access control-node execute
```
vagrant ssh controller
```

### Stopping the machines
```
vagrant halt
```

### Starting the machines back again
```
vagrant up
```

## Hello world
In order to test if everything works correctly an ad-hoc command can be used
1. SSH to the control node from your host machine
```
vagrant ssh controller
```

2. Create a project directory
```
mkdir -p helloworld
```
3. Navigate to the project's directory
```
cd helloworld
```

4. Create an inventory file
```
echo "[servers]" > inventory
for i in {1..4}; do
  echo managed$i >> inventory
done
```

5. Install ansible
```
sudo yum install -y ansible
```

6. Execute ping command which checks connectivity
```
ansible localhost,servers -i inventory -m ping
```
You should see something similar to the below output, it's an indication of correctly working infrastructure
```
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
managed1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
managed3 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
managed4 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
managed2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

## Inspirations
* [RHCE_ENV](https://github.com/theJaxon/RHCE_ENV)
* [ansible8env](https://github.com/rdbreak/ansible8env)
