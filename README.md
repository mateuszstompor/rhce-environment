# RHCE environment
This repository contains a vagrant configuration which describes infrastructure of nodes that can be used for RHCE exam preparation (EX294). Idea of creating such a similiar project in comparision to what's currently available is to provide set of exercises and solutions which will be relevant to the configuration and allow the people preparing for the exam to check exact steps they need to follow in order to solve them

## Exercises 
Look at [this repository](https://github.com/mateuszstompor/rhce-ex294-exam). More exercises are going to be added soon 

## Assumptions
Vagrant file has description of a minimal installation and infrastructure description. There are five nodes in total. One control node - controller - and four managed nodes - host names `managed[1:4]`. Managed hosts can be accessed from controller passwordless using the root account. **Root password is `vagrant`**

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
In order to access control node execute
```
vagrant ssh controller
```

### Stopping the machines
```
vagrant halt
```

### Start the machines back again
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

6. Execute ping command which check connectivity
```
ansible localhost,servers -i inventory -m ping
```
You should see something similar to below output, it's an indication of correctly working infrastructure
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
