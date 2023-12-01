# installation steps 
here we have 3 server nodes. one is master and other 2 are worker nodes
These commands should be run in the master node
```
sudo apt update
sudo apt install ansible
sudo apt install sshpass
```
edit the .yml and .ini files

```
Create an inventory file named inventory.ini:


[master]
192.168.138.111 ansible_user=your_username ansible_ssh_pass=your_ssh_password

[workers]
192.168.138.222 ansible_user=your_username ansible_ssh_pass=your_ssh_password
192.168.138.223 ansible_user=your_username ansible_ssh_pass=your_ssh_password
```
```Replace your_username and your_ssh_password with your actual SSH username and password for the PCs.```


## add the other pc to the known hosts in the master node
```
  ssh-keyscan 192.168.138.111 >> ~/.ssh/known_hosts
  ssh-keyscan 192.168.138.222 >> ~/.ssh/known_hosts
  ssh-keyscan 192.168.138.223 >> ~/.ssh/known_hosts

```
```OR```
## Use SSH Key-Based Authentication

If you prefer using SSH keys for authentication, follow these steps:
a. Generate SSH Key Pair

On your Ansible control machine, generate an SSH key pair:

bash
```
ssh-keygen -t rsa
```
b. Copy Public Key to Target Hosts

Copy the public key to each target host:

bash
```
ssh-copy-id idrbt@192.168.138.111
ssh-copy-id idrbt@192.168.138.222
ssh-copy-id idrbt@192.168.138.223
```
c. Update inventory.ini to Use SSH Key

Modify your inventory.ini to use SSH key instead of password:

ini
```
[master]
192.168.138.111 ansible_user=idrbt ansible_ssh_private_key_file=/path/to/id_rsa

[workers]
192.168.138.222 ansible_user=idrbt ansible_ssh_private_key_file=/path/to/id_rsa
192.168.138.223 ansible_user=idrbt ansible_ssh_private_key_file=/path/to/id_rsa
```
Replace /path/to/id_rsa with the actual path to your private SSH key.
Retry Ansible Playbook

After making the changes, retry running the Ansible playbook:

bash
```
ansible-playbook -i inventory.ini docker-setup.yml
```
# Running the playbook file
for docker installation
```
  ansible-playbook -i inventory.ini docker-setup.yml

```
For K8's multi node setup installation
```
ansible-playbook -i inventory.ini k8s-setup.yml

```

For
