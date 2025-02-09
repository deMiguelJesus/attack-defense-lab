We will create:

1. **Kali Linux VM** (Attacker machine) – Used for penetration testing.
2. **Metasploitable 2 VM** (Target machine) – A deliberately vulnerable Linux system.

Both VMs will be **isolated from the internet** and connected through a private network.



Steps

Step 1: Install KVM and Required Tools

KVM (Kernel-based Virtual Machine) – Built into the Linux kernel, KVM is the best choice for running VMs on Linux servers. It's powerful and used in enterprise environments.

`libvirt` is a **virtualization management toolkit** that provides a **consistent API** for managing different hypervisors, including **KVM**, **QEMU**, **Xen**, and **LXC**. It is commonly used to manage **virtual machines (VMs)** on Linux.

When you install `libvirt`, it runs a background service called `libvirtd`, which handles VM creation, management, networking, and storage.

✅ **Includes a CLI (`virsh`) and GUI (`virt-manager`)** – Manage VMs via command line or graphical interface.


```
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt,kvm $USER
```


Step 2: Download ISO/VM Images

[https://www.kali.org/docs/virtualization/install-qemu-guest-vm/](https://www.kali.org/docs/virtualization/install-qemu-guest-vm/)

[https://docs.rapid7.com/metasploit/metasploitable-2/](https://docs.rapid7.com/metasploit/metasploitable-2/)

Default credentials: msfadmin/msfadmin

  

Step 3: Create a Private Network. To allow communication between the two VMs but block internet access, create a virtual network.


To allow communication between the two VMs but **block internet access**, create a virtual network.

```
sudo nano /etc/libvirt/qemu/networks/lab-network.xml
```

```
<network>
    <name>lab-network</name>
    <bridge name='virbr1' />
    <forward mode='nat'/>
    <ip address='192.168.56.1' netmask='255.255.255.0'>
        <dhcp>
            <range start='192.168.56.100' end='192.168.56.200' />
        </dhcp>
    </ip>
</network>
```

define and restart the network

```
sudo virsh net-define /etc/libvirt/qemu/networks/lab-network.xml
sudo virsh net-autostart lab-network
sudo virsh net-start lab-network
```

Verify
```
virsh net-list --all
```

```
 Name          State    Autostart   Persistent
------------------------------------------------
 default       active   yes         yes
 lab-network   active   yes         yes
```


🔷 Step 4: Create and Configure Kali Linux VM

    Open Virt-Manager:

virt-manager

Click "Create a new virtual machine" and select "Local install media (ISO)".
Select the Kali Linux ISO you downloaded.
Allocate resources:

    RAM: 4GB (4096 MB)
    CPU: 2 vCPUs

Create a 20GB disk.
Network settings:

    Choose "Specify shared device name" and select lab-network.

Finish the installation and complete the Kali setup.



Step 5: Create and Configure Metasploitable 2 VM

Since Metasploitable 2 is a pre-configured VM, we will import it.

    Extract the Metasploitable2 ZIP file:

unzip Metasploitable2.zip -d ~/VMs/

Convert the VMDK disk to QCOW2 format:

qemu-img convert -f vmdk -O qcow2 ~/VMs/Metasploitable.vmdk ~/VMs/metasploitable2.qcow2

Create a new VM in virt-manager:

    Select "Import existing disk image".
    Choose the metasploitable2.qcow2 file.
    Set RAM to 1GB, CPU to 1 vCPU.
    Set Network to lab-network.
    Complete the process and start the VM.




🔷 Step 6: Test Network and Setup
🔹 Check VM IP Addresses

Run on both VMs:

ip a

Both should have 192.168.56.x IPs.
🔹 Check Connectivity

From Kali, test connection to Metasploitable:

ping 192.168.56.101

