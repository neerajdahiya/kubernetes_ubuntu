## Create VM in Virtual box

**Using Virtualbox UI create 3 VMs.**

**Some useful tips:**

*- Ensure to provide sufficient storage to each VM*
*- To preserve size choose minimal install option*
*- Name VMs so that it can be easily identified in Virtualbox UI. Like master01_cluster01, worker01_cluster01, etc.*
*- provide username such that you can easily identify master / worker nodes. Ex: for master01_cluster01 VM give username as master01, and for worker01_cluster01 as worker01*
***- Use 'Bridged Adapter' under Network options for all VMs. This would allow external DHCP server (ex: router on LAN) to assign IPs to Virtual Machine. Additionally HOST can also be used to deploy applications outside cluster (scenarios like CICD can easily be tested with this setup if HOST has limited resources)***


## Enable SSH on all Virtual Machines

### Install SSH server
> sudo apt install openssh-server

**-	Use ssh from host computer to connect to all machines and execute steps in next points.
	-	This is quite important as if each machine is configured via local terminal then it would be lot of manual overhead (as can't copy data between machines). Also worker node addition would require master node kubeadm command to be executed on all machines.**

### Enable Password Authentication
> vi sshd_config
>   -	uncomment PasswordAuthentication yes
