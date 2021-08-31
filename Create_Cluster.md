# Create VMs

**Using Virtualbox to create 3 VMs.**

**Some useful tips:**
- *Ensure to provide sufficient storage to each VM*
- *To preserve size choose minimal install option*
- *Name VMs so that it can be easily identified in Virtualbox UI. Like master01_cluster01, worker01_cluster01, etc.*
- *Provide username such that you can easily identify master / worker nodes. Ex: for master01_cluster01 VM give username as master01, and for worker01_cluster01 as worker01*
- ***Use 'Bridged Adapter' under Network options for all VMs. This would allow external DHCP server (ex: router on LAN) to assign IPs to Virtual Machine. Additionally HOST can also be used to deploy applications outside cluster (scenarios like CICD can easily be tested with this setup if HOST has limited resources)***
- ***Create VM snapshot immediately after clean OS install, this can act as base for clone in case there is need to spawn new VMs (to avoid OS installation steps)***

## Install net-tools
*Required to check interface ip and mac*
> sudo apt install net-tools

## Enable SSH on all Virtual Machines
*Required to be completed by logging in to individual Virtual Machines*

### Install SSH server
> sudo apt install openssh-server

- **Use ssh from host computer to connect to all machines and execute steps in next points.**
	-This is quite important as if each machine is configured via local terminal then it would be lot of manual overhead (as can't copy data between machines). Also worker node addition would require master node kubeadm command to be executed on all machines.

### Enable Password Authentication
> vi sshd_config
> 	- uncomment PasswordAuthentication yes

### Restart ssh service
> sudo systemctl restart sshd.service

## (Optional) Enable FTP server
*FTP can be enabled on VMs in case file transfer is required between Host -- Guest / Guest -- Guest.*

### Install FTP server
> sudo apt install vsftpd


# Install Docker
- Refer https://docs.docker.com/engine/install/ubuntu/ for official documentation

### Uninstall old versions
> sudo apt-get remove docker docker-engine docker.io containerd runc

### Update the apt package index and install packages to allow apt to use a repository over HTTPS
> sudo apt-get update
> sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release

### Add Docker’s official GPG key
> curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

### Set up the stable repository
> echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
