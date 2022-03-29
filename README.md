# how to create vm desktop linux and accessing it through novnc

## Create the VM
1. Download ubuntu cloud image, from https://cloud-images.ubuntu.com/
2. copy the image file and resize the disk image, for example
    
        cp ~/image/focal-server-cloudimg-amd64.img ubuntu1.img
        qemu-img resize ubuntu1.img 40G
    
3. start VM using the following script

        #!/bin/bash
        VM=ubuntu1
        DISK=${VM}.img
        CDROM=seed.img
        virt-install --name ${VM} \
          --disk ./${DISK},device=disk,bus=virtio \
          --disk ./${CDROM},device=cdrom \
          --ram 4096 --vcpu 1  \
          --os-type linux --os-variant generic \
          --network bridge=lan1,model=virtio \
          --console pty,target_type=serial \
          --noautoconsole \
          --hvm --accelerate  \
          --graphics vnc,port=5901,listen=0.0.0.0  \
          --virt-type=kvm  \
          --boot hd

4. access the console of the VM
  
        virsh console ubuntu1

5. Do the following to update the system and install xfce4 desktop

        sudo apt -y update && sudo apt -y upgrade
        sudo apt -y install xfce4 firefox
        sudo apt -y autoremove lightdm
    
6. Do the following to setup the desktop environment

        echo "[User]
        Session=
        XSession=xfce
        Icon=/home/ubuntu/.face
        SystemAccount=false

        [InputSource0]
        xkb=us" | sudo tee  /var/lib/AccountsService/users/ubuntu

7. Do the following to setup the network configuration to NetworkManager under the desktop 

        sudo  rm /etc/netplan/*
        echo "network:
            version: 2
            # renderer: networkd
            renderer: NetworkManager
        " | sudo tee /etc/netplan/01_net.yaml
    
## Installing novnc and enable access to Linux VM video console through web interface

1. On the hypervisor, install novnc software

        sudo apt install novnc

2. Run the following start access through novnc

        websockify -D --web=/usr/share/novnc 6801 127.0.0.1:5901

        6801 : the novnc port
        5901 : the vnc port for the VM 

3. To automatically start novnc, the  command above can be added into startup script of the hypervisor

## accessing Linux VM video console through web interface
From your workstation, use the web browser to access the video console of the VM, and use the following URL

    http://<IP_address_of_hypervisor>:<novnc_port>/vnc.html
    
    for example
    
    http://192.168.5.101:6801/vnc.html
    
 
    
