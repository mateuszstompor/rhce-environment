# RHCE environment
This repository contains a vagrant configuration which describes infrastructure of nodes that can be used for RHCE exam preparation (EX294). Idea of creating such a similiar project in comparision to what's currently available is to provide set of exercises and solutions which will be relevant to the configuration and allow the people preparing for the exam to check exact steps they need to follow in order to solve them.

## Exercises 
Stay tuned...

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


## Inspirations
* [RHCE_ENV](https://github.com/theJaxon/RHCE_ENV)
* [ansible8env](https://github.com/rdbreak/ansible8env)