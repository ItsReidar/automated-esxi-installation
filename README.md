## Automated installation of VMware ESXi 7.0 via a tftp server, with kickstart
This configuration is done on a **OpenSUSE server**. Steps can also be applied on other linux distros.


## TFTP Server
Start with installing the TFTP server
```bash
sudo zypper install tftp yast2-tftp-server -y
```
>You can configure the tftp server trough YaST but we'll do a manual configuration of the server.

Next start the service and enable it (so it starts at boot).
```bash
sudo systemctl enable tftp.socket
```
```bash
sudo systemctl start tftp.socket
```
**Make sure** to let UDP port 69 trough in the firewall, or just disable the firewall on the machine **(not recommended)**.

For the tftp config look in to the **tftp config in the repo**!


## DNSMASQ
Next we install the dnsmasq for dhcp server.
```bash
sudo zypper install dnsmasq dnsmasq-utils -y
```
Again start the service and enable it
```bash
sudo systemctl enable tftp.socket
```
```bash
sudo systemctl start tftp.socket
```
Again look in the repo for the **dnsmasq config.**


## SYSLINUX
Next on the list is syslinux. First install syslinux.

```bash
sudo zypper install syslinux -y
```

Then copy everything from the syslinyx directory to the tftpboot directory.

```bash
cp -r /usr/share/syslinux/* /srv/tftpboot
```

## HTTP Server
Now we install a http server to host our kickstart files later.

```bash
sudo zypper install apache2 -y
```
```bash
sudo systemctl enable apache2
```
```bash
sudo systemctl start apache2
```


## Copying files to TFTP and configuration
Now we will prepare a installation directory for the ESXi files.

First make a directory which we can mount our ISO file (CD) to.

```bash
mkdir /mnt/VMware-VMvisor-Installer-7.0U1c-17325551.x86_64.iso
```

Then mount the ISO to this directory.
```bash
mount /dev/sr0 /mnt/esxi7
```
or (depends on underlying kernel)
```bash
mount /dev/cdrom /mnt/esxi7
```


Make a directory in the **tftpboot directory** with a **logical name** (ex. esxi7). This is where we copy the esxi files to and will boot from trough pxe.
```bash
mkdir /srv/tftpboot/esxi7
```
Then copy all the files from the ISO to the directory on the server
```bash
cp -rf /mnt/esxi7 /srv/tftpboot/esxi7
```

If the ISO refuses to mount do the same steps but make the name mount directory exactly the name of the ISO file.
```bash
mkdir /mnt/VMware-VMvisor-Installer-7.0U1c-17325551.x86_64.iso
mount /dev/sr0 /mnt/VMware-VMvisor-Installer-7.0U1c-17325551.x86_64.iso
mkdir /srv/tftpboot/esxi7
cp -rf /mnt/VMware-VMvisor-Installer-7.0U1c-17325551.x86_64.iso/* /srv/tftpboot/esxi7/
```

Don't forget to unmount the ISO.
```bash
umount / mnt/VMware-VMvisor-Installer-7.0U1c-17325551.x86_64.iso
```

Next in the /srv/tftpboot/esxi7 directory we will remove all unecessary "/" from the boot.cfg and efi boot.cfg files.
```bash
sed -i 's#/##g' /srv/tftpboot/esxi7/boot.cfg
```
```bash
sed -i 's#/##g' /srv/tftpboot/esxi7/efi/boot/boot.cfg
```
Delete the contents of the file and paste the one's from the config in the repo.

We also will change 1 option in the /srv/tftpboot/esxi7/boot.cfg file.
* prefix=name-of-your-esxi-istallation-folder

**DO NOT CHANGE ANYTHING ELSE**


### PXE Boot menu
Next we'll make the PXE Boot menu. This can be used if you configure multiple installers on the server.
```bash
mkdir /srv/tftpboot/pxelinux.cfg
```
```bash
touch /srv/tftpboot/pxelinux.cfg/default
```

### Kickstart Configuration
```bash
mkdir /srv/www/htdocs/ks
```
```bash
touch /srv/www/htdocs/ks/esxi7.cgf
```
Then add the config included in this repo and change according to your configuration.

Voila, now your connected system should be able to automatically start a install of esxi 7.0 and configure itself.
