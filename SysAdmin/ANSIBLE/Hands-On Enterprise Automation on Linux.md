# Hands-On Entreprise Automation on Linux

---

**1) Building a SOE on Linux**

---

Global configuration such as sshd, monitor and such should be baked in the original image (**Gold Build**) as it is inefficient to deploy them each time we roll-out a VM.

Gold Build should be updated and maintened regularly.

---

**2) Automating with Ansible**

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

