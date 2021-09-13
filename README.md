# Week--8-Ansible-IAC-Terraforming-
```
Questions:
what are play books ?
 

- Configuration management (with ansible): managment as a push vagrant 

ocestration (with terraform): 

- Push vs Pull configuration managment
- which tool are used for push and pull configurations ?
Difference btween:configuration and ocestration?
- What is ansible ?
- what is terraforming ?

- creat a diagram for ansible on prem, hybrif and public arthitecture ?

- What is the default directory structure for ansible ? 
- What is the Inventory/hosts file and the purpose of it?

- What should be added to host file to establish a secure connection btween ansible controller and agent controller ?
- What are ad-hoc commands ?

- Add a struture of creating adhoc commands `ansible all -m ping`

```



## Topic Covered 
- Ansible 
- IOC (Infrastructure as code )
- Yaml and playbooks 
____

Goal:
try and connect to mutiple instances with one controller  instance 

![image](https://intellipaat.com/mediaFiles/2018/12/8.png)

# Ansible controller and agent nodes set up guide
- Clone this repo and run `vagrant up`
- `(double check syntax/intendation)`

## We will use 18.04 ubuntu for ansible controller and agent nodes set up 
### Please ensure to refer back to your vagrant documentation

- **You may need to reinstall plugins or dependencies required depending on the OS you are using.**

```vagrant 
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what

# MULTI SERVER/VMs environment 
#
Vagrant.configure("2") do |config|

# creating first VM called web  
  config.vm.define "web" do |web|
    
    web.vm.box = "bento/ubuntu-18.04"
   # downloading ubuntu 18.04 image

    web.vm.hostname = 'web'
    # assigning host name to the VM
    
    web.vm.network :private_network, ip: "192.168.33.10"
    #   assigning private IP
    
    # config.hostsupdater.aliases = ["development.web"]
    # creating a link called development.web so we can access web page with this link instread of an IP   
        
  end
  
# creating second VM called db
  config.vm.define "db" do |db|
    
    db.vm.box = "bento/ubuntu-18.04"
    
    db.vm.hostname = 'db'
    
    db.vm.network :private_network, ip: "192.168.33.11"
    
    
    #config.hostsupdater.aliases = ["development.db"]     
  end

 # creating are Ansible controller
  config.vm.define "controller" do |controller|
    
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
    #config.hostsupdater.aliases = ["development.controller"] 
    
  end

end
```

Type `vagrant status` in your terminal to check if all 3 vm's are running \
You should see this: 

```
web                       running (virtualbox)
db                        running (virtualbox)
controller                running (virtualbox)
```

___

## Setting up The vagrant files 
### Vagrant up on the file and run these commands

```
sudo apt-get update -y
sudo apt-get upgrade -y
```

ping 192.168.33.[10(app) , 11(DB)] to check that the DB and WEB is up and avaliable.


___

SSH into the controller vm 

and type 
```
- sudo apt-get install software-properties-common -y 

- sudo apt-add-repository ppa:ansible/ansible -y 

sudo apt-get install ansible -y

ansible --version 
```
## Now the controller vm is able to control the other other vm's , but before we need to create a SSH into the vm's


SSHing into the WEB app:
```
ssh vagrant@192.168.33.[10 or 11]
type for passoword: yes
password:vagrant
exit to return back to the controller 
```

cd /ect/ansible
enter the hosts file via sudo nano

add the Web ip and save it under [web]

```
192.168.33.10 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant  

192.168.33.11 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant 
```
In bash Type: \
`ansible [web or web] -m ping` \
you should get a green results with result = "PONG"

## Adhoc Commands:
Any linux you can run on the vm's can be run in the controller without having to ssh into the vms's e.g. saving time 

```
ansible all -a "uname" - list the name of the vm"

ansible [all,web,db ]-a "ls -a" - show the folders in the current dir
 ```

___

# Ansible Playbooks
- Ansible playbooks are a different way of using ad-hoc tasks.
 - used for installing mutiple commands on an agent server 
 - playbooks are .yml or .ymal files. 
 - playbooks start with `---` 3 dahses and like python use indents  


###  playbook.yml file

![image](https://s3.amazonaws.com/media-p.slid.es/uploads/racku/images/297626/Ansible_Playbook.png)



```
# create a playbook to install nginx web server on a web machine 
# web 192.168.33.10
# now with these 3 dashes it becomes a YAML file
---
# add the name of the host
- hosts: web
# gather facts 
  gather_facts: yes
# now we need admin access 
  become: true
# add instructions to install nginx on web machine 
# state present means that the package is running 
  tasks: 
  - name: Install Nginx
    apt: pkg=nginx state=present
# ensure the nginx server is running 

```

Create an `nginx_playbook.yml` in the controller VM's `/etc/ansible` directory:
  - In the same directory run this command 
```
sudo ansible-playbook nginx_playbook.yml
```
To check if the command correctly installed the command type this:

```
ansible web -a "sudo systemctl status nginx"
```
you should see this Meaning that the nginx is running.
```
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2021-09-13 13:28:26 UTC; 4min 26s ago
     Docs: man:nginx(8)
 Main PID: 24089 (nginx)
    Tasks: 3 (limit: 1104)
   CGroup: /system.slice/nginx.service
           ├─24089 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           ├─24091 nginx: worker process
           └─24092 nginx: worker process
```
