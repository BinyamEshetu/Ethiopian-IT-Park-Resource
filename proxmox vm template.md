In some cases, tutorials will instruct you to use libguestfs-toolsto inject the agent into your cloud-init image.  The problem with this, is that it sets a machine-id that will then get copied over to all VMs you end up creating based off of your the template.  This, in my experience, can become problematic fairly quickly. Especially with DHCP/networking confusion.
## Initial steps
- create a vm with basic configurations
    - id: 500
    - name: ubuntu-server-focal
    - os: do not use any media
    - type: linux
    - system: qemu agent = yes
    - disk: remove scsi0 disk
    - cpu: 4 core
    - memory: 4096
    - network: vmbr0
- go to hardware and add a cloud init drive
    - bus: ide0
    - storage: your vm storage
- go to cloud-init tab
    - user: root
    - password: 
    - ip config: ipv4 and ipv6 dhcp
    - then click on regenerate

## Command line
- log into your proxmox through cli
- download the cloud images you want to create a template for
    - wget https://cloud-images.ubuntu.com/releases/jammy/release/ubuntu-22.04-server-cloudimg-amd64.img
- convert the downloaded file extenstion to qcow2
    - mv ubuntu-22.04-server-cloudimg-amd64.img ubuntu-22.04-server.qcow2
- enable serial console for the vm
    - qm set 500 --serial0 socket --vga serial0
- resize your cloud image
    - qemu-img resize ubuntu-22.04-server.qcow2 10G
- import the disk to your vm
    - qm importdisk 500 ubuntu-22.04-server.qcow2 VM_store

## final configurations
- go to the hardware section of your vm and click on the Unused disk0 entry
    - tick on ssd emulation
    - cache to write back
    - discard to yes
    - then click on add
- go to the option tab of your vm
    - click on the boot order
    - tick on the scsi0 entry and move it the the top
    - click on ok

## installing and configuring additional options
- powerup the vm
    - sudo apt update && sudo apt upgrade -y && sudo apt install qemu-guest-agent
    - sudo systemctl enable qemu-guest-agent
    - cat /dev/null > /etc/machine-id
    - sudo nano /etc/ssh/sshd_config
        - enable permitrootlogin to yes
        - enable passwordauthentication yes
    - cloud-init clean
    - shutdown now

## convert to template
- right click on the vm and clone it to make a backup incase we need to make additional configurations later on
- then right click on the vm and convert to template

# template from iso(desktop template)

- Install the ubuntu desktop vm as we always do
- once installation