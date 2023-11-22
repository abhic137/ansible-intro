# ansible-intro
 To automate these commands across multiple PCs using Ansible, you'll need to follow these steps:

1. **Install Ansible:**
   Make sure Ansible is installed on the machine from which you will run the commands. You can install Ansible using the package manager of your choice. For example, on Ubuntu, you can use:

   ```bash
   sudo apt update
   sudo apt install ansible -y
   ```

2. **Create an Ansible Inventory File:**
   Create an inventory file that lists the IP addresses or hostnames of the five PCs. Save this file as `inventory.ini`:

   ```ini
   [pcs]
   pc1 ansible_host=pc1_ip_address
   pc2 ansible_host=pc2_ip_address
   pc3 ansible_host=pc3_ip_address
   pc4 ansible_host=pc4_ip_address
   pc5 ansible_host=pc5_ip_address
   ```

   Replace `pc1_ip_address`, `pc2_ip_address`, etc., with the actual IP addresses of your PCs.

3. **Write Ansible Playbook:**
   Create an Ansible playbook, let's say `setup.yml`, with the following content:

   ```yaml
   ---
   - name: Setup Docker and Kubernetes
     hosts: pcs
     become: true
     tasks:
       - name: Update package repositories
         apt:
           update_cache: yes

       - name: Install required packages
         apt:
           name: "{{ item }}"
           state: present
         loop:
           - apt-transport-https
           - curl

       - name: Create directory /etc/apt/keyrings
         file:
           path: /etc/apt/keyrings
           state: directory

       - name: Download Docker GPG key
         command: curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

       - name: Add Docker repository
         lineinfile:
           path: /etc/apt/sources.list.d/docker.list
           line: "deb [arch={{ dpkg_architecture }} signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu {{ lsb_codename }} stable"
           create: yes
         vars:
           dpkg_architecture: "{{ dpkg --print-architecture }}"
           lsb_codename: "{{ lsb_release -cs }}"

       - name: Update package repositories after adding Docker repository
         apt:
           update_cache: yes

       - name: Install containerd.io
         apt:
           name: containerd.io
           state: present

       - name: Create directory /etc/containerd
         file:
           path: /etc/containerd
           state: directory

       - name: Configure containerd
         command: containerd config default > /etc/containerd/config.toml

       - name: Add Kubernetes GPG key
         command: curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add

       - name: Add Kubernetes repository
         apt_repository:
           repo: "deb http://apt.kubernetes.io/ kubernetes-xenial main"
           state: present

       - name: Install Kubernetes packages
         apt:
           name: "{{ item }}"
           state: present
         loop:
           - kubeadm
           - kubelet
           - kubectl
           - kubernetes-cni

       - name: Disable swap
         command: swapoff -a

       - name: Load br_netfilter kernel module
         command: modprobe br_netfilter

       - name: Enable IP forwarding
         sysctl:
           name: net.ipv4.ip_forward
           value: 1
           state: present
           sysctl_set: yes
     ```

4. **Run the Ansible Playbook:**
   Run the Ansible playbook using the following command:

   ```bash
   ansible-playbook -i inventory.ini setup.yml
   ```

   This command will execute the tasks on all the PCs listed in the inventory file.

Ensure that SSH access is set up between the machine running Ansible and the target PCs. You might need to provide the SSH key or password for authentication.

This playbook assumes that the target machines are running Ubuntu. If you are using a different Linux distribution, you might need to adjust package names and commands accordingly.
