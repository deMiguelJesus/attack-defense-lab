# 1. 1. Install KVM and Required Tools

KVM (Kernel-based Virtual Machine) is built into the Linux kernel, KVM is the best choice for running VMs on Linux servers. It's powerful and used in enterprise environments.

`libvirt` is a **virtualization management toolkit** that provides a **consistent API** for managing different hypervisors, including **KVM**, **QEMU**, **Xen**, and **LXC**. It is commonly used to manage **virtual machines (VMs)** on Linux. When you install `libvirt`, it runs a background service called `libvirtd`, which handles VM creation, management, networking, and storage. **Includes a CLI (`virsh`) and GUI (`virt-manager`)** – Manage VMs via command line or graphical interface.

```
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt,kvm $USER
```


## 1.2. Create a Private Network

To allow communication between the two VMs but **block internet access**, create a virtual network. In this example, it will called `lab-network`: 

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

Define and restart the network:

```
sudo virsh net-define /etc/libvirt/qemu/networks/lab-network.xml
sudo virsh net-autostart lab-network
sudo virsh net-start lab-network
```

Verify:
```
virsh net-list --all
```

```
 Name          State    Autostart   Persistent
------------------------------------------------
 default       active   yes         yes
 lab-network   active   yes         yes
```


## 1.3. Install ISO/VM Images

Open Virt-Manager:

```
sudo virt-manager
```

### 1.3.1. Kali Linux

- ISO Image: https://cdimage.kali.org/kali-2024.4/kali-linux-2024.4-installer-amd64.iso
- Official guide [HERE](https://www.kali.org/docs/virtualization/install-qemu-guest-vm/)

- Click "Create a new virtual machine" and select "Local install media (ISO)".
- Select the Kali Linux ISO you downloaded. Allocate resources:
```
    RAM: 4GB (4096 MB)
    CPU: 2 vCPUs
```

- Create a 20 GB disk.
- Network settings:

    Choose "Specify shared device name" and select lab-network.

- Finish the installation and complete the Kali setup (All by default).

### 1.3.2. Metasploitable 2

- VM Image: [https://docs.rapid7.com/metasploit/metasploitable-2/](https://docs.rapid7.com/metasploit/metasploitable-2/)
- Default credentials: msfadmin/msfadmin

Since Metasploitable 2 is a pre-configured VM, we will import it. Extract the Metasploitable2 ZIP file:

```
unzip Metasploitable2.zip -d ~/VMs/
```

Convert the VMDK disk to QCOW2 format:
```
qemu-img convert -f vmdk -O qcow2 ~/VMs/Metasploitable.vmdk ~/VMs/metasploitable2.qcow2
```

Create a new VM in virt-manager:

- Select "Import existing disk image".
- Choose the metasploitable2.qcow2 file.
- Set RAM to 1GB, CPU to 1 vCPU.
- Set Network to lab-network.
- Complete the process and start the VM.

## 1.3. Test network and setup

Check VM IP Addresses by running on both VMs: 

```
ip a
```

Both should have 192.168.56.x IPs.

Check Connectivity by sending a ping from Kali:

```
ping <ip-metasploitable2>
```

