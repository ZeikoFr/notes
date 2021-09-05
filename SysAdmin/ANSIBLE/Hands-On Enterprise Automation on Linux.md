# Hands-On Entreprise Automation on Linux

---

### Building a SOE on Linux

---

Global configuration such as sshd, monitor and such should be baked in the original image (**Gold Build**) as it is inefficient to deploy them each time we roll-out a VM.

Gold Build should be updated and maintened regularly.

---

### Automating with Ansible

---

First thing first install :

Ubuntu 18.04 LTS (20 is available at time of writing but I preferred to stick with the book version)
Ansible V2.9.24
```bash
sudo apt update  
sudo apt install software-properties-common  
sudo apt-add-repository --yes --update ppa:ansible/ansible  
sudo apt-get install ansible
```

Now that ansible is installed we can dive in the configuration :

`/etc/ansible/ansible.cfg` Default conf file, we are not modifying it for now.

To work Ansible need an inventory. A sample file is usually here at the install of ansible : `/etc/ansible/hosts`

When empty ansible implicitly operates against localhost only.

A simple playbook :

```yaml
---
- name: SimplePlaybook
  hosts: localhost
  become: false
```

Ansible YAML always start with "---" and the sigle dash "-" denote the start of the play. Name is optionnal but streongly recommended

**host** indicate which host the playbook is performing on, here is localhost

**become** specify if ansible need to run as root or not.

Now we need to give action to the playbook :

```yaml
---  
- name: Simple Playbook  
 hosts: localhost  
 become: false  
 tasks:  
   - name: Show a message  
     debug:  
       msg: "Hello world"  
  
   - name: Touch a file  
     file:  
       path: /tmp/foo  
       state: touch
```

**tasks** define end of the play definition and the actual start of the task

**debug** as the name, used for debugging here used to say "this is the message we want to show"

**file** module to manipulate file, here we create a file named foo with the absolute location.

---

### Exploring inventories with ansible

---

Inventory can be static or dynamic.

Using a static inventory I create a group with 2 hosts

```bash
[test]
host1
host1
```

To simplify the work I've manually added the ip and name of each host in /etc/hosts

now I can push the playbook to both of by modifying `hosts: localhost`to `hosts: all` in my previous playbook

to push the playbook with an password based authentication i first need to modify the value of `host_key_checking` in /etc/ansible/ansible.cfg

Now I can push my playbook : `ansible-playbook -i hosts --ask-pass simpleplaybook.yaml`

To use a key based auth I simply generated an unencrypted rsa key and pushed it to my host with **ssh-copy-id**

Here I use my current user that is present on all the VM to push the configuration, but ansible is flexible enough to set the user per group or per host.

---

### Understanding roles in Ansible

---

We're going to start with a simple install of mariadb whiout configuration :

```yaml
---  
- name: Install MariaDB Server  
 hosts: localhost  
 become: true  
  
 tasks:  
   - name: Install mariadb-server package  
     apt:  
       name: mariadb-server  
       update_cache: yes
```

As this is executed on the local machine I needed to execute the playbook with sudo.

Now I've created this arborescence : roles/install-mariadb/tasks/main.yml

Now in my playbook file **located in the same folder as the directory "roles"** : 

```yaml
---  
- name: Install MariaDB Server  
 hosts: localhost  
 become: true  
  
 roles:  
  - install-mariadb
```

Here there's a confusion that can be made, **"roles"** define the *folder* named **roles** and **install-mariadb** the subfolder with the same name.

So basically the playbook here is simply looking in **roles/install-mariadb/** then found the folder **task** than contain the **main.yml** where the action is defined.


##### Using Ansible Galaxy

Here we simply DL and execute this `ansible-galaxy install -p roles/ mrlesmithjr.mariadb-mysql`

 This playbook install and configure mariadb
 
 This part is just to see how easy it is to run "complex" playbook

---

### Understanding Ansible variables

---

There's a lot of things variables can do in ansible, but we need to be carefull as there's a strict order of precedence.

 To start, I just declare a variable that will be executed in the same file.
 
 ```yaml
---
- name: Simple Playbook  
 hosts: localhost  
 become: false  
  
 vars:  
   message: "Life is beautiful!"  
  
 tasks:  
   - name: Show a message  
     debug:  
       msg: "{{ message }}"  
  
   - name: Touch a file  
     file:  
       path: /tmp/foo  
       state: touch
 ```
 
If we want to overwrite the variable that's in the playbook we can following the varaible precedence list, simply put the messsage as a variables passed in the ansible-playbook command as it is the top of the list and override any other variable :

`simpleplaybook.yaml -e "message=\\"Hello from CLI\\""`

There's a wild array of ansible variable with key system data.

To saw all the gathered variable from ansible : `ansible -m setup localhost`

For example if you want to show the OS used :

```yaml
   - name: Show a message  
     debug:  
       msg: "{{ ansible_distribution }}"
```

When passing sensitive info in ansible Playbook **Vault** should be used.

---

### Understanding Ansible templates

---

Ansible template can help to easier configuration file deployment.

We could for that use regex and replace in the playbook but it is inefficient, slower and prone to error.

 But ansible use a technology called Jinja2 Templating coming from python.
 
 Now to define a virtual host as a template (for example)
 
```bash
<VirtualHost *:80>
    DocumentRoot {{ docroot }}
    ServerName www.example.com
</VirtualHost>
```

Jinja2 is capable of conditional statement and loop, and a wild array of filter.

---

### Streamlining Infrastructure Management with AWX

---

installation of AWX :

Modern installation of AWX require AWX-Operator a Kubernetes operator for AWX.

For testing and dev purpose AWX work with Docker container so we need to install docker first :

```bash
sudo apt-get install git docker.io python-docker docker-compose  
git clone https://github.com/ansible/awx.git
```

So since AWX require a K8S cluster to be install and I don't want to go this route I choosed to use semaphore :

https://computingforgeeks.com/install-semaphore-ansible-web-ui-on-ubuntu-debian/