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