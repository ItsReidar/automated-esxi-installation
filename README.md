## Automated installation of ESXi 7.0U3 (DNSMASQ, TFTP, HTTP, SYSLINUX, KICKSTART)
This configuration is done on a **OpenSUSE server**. Steps can also be applied on other linux distros.

### Prerequisites

Let's begin with installing the needed services. We will need a dhcp forwarder, tftp-server, syslinux and a webserver.

Install following packages through Yast (Recommended):
- tftp
- yast2-tftp-server
- dnsmasq
- syslinux
- apache2

or through the command line.
```bash
sudo zypper install tftp yast2-tftp-server dnsmasq dnsmasq-utils syslinux apache2 -y
```

### DHCP
Assuming there's a DHCP server on the network we will just configure a dhcp forwarder. The config file can be found under /etc/dnsmasq.conf

Next enable and start the service:
```bash
sudo systemctl enable dnsmasq
```
```bash
sudo systemctl start dnsmasq
```
### Installing the webserver
Next allow through thefirewall, enable and start the webservice:
```bash
sudo firewall-cmd --zone=public --add-service=http --permanent && firewall-cmd --reload
```
```bash
sudo systemctl enable apache2
```
```bash
sudo systemctl start apache2
```
```bash
chmod -R 755 /srv/www/htdocs/ks && chmod -R 755 /srv/www/htdocs/iso-files
```

### TFTP
Configure the TFTP service. The config file can be found under /etc/xinetd.d/tftp.
>Fill it with the config from the repo.

Next enable and start the service:
```bash
sudo systemctl enable tftp.socket
```
```bash
sudo systemctl start tftp.socket
```
```bash
sudo systemctl enable xinetd
```
```bash
sudo systemctl start xinetd
```
And add it to the firewall:
```bash
sudo firewall-cmd --zone=public --add-service=tftp --permanent
```
```bash
sudo firewall-cmd --reload
```

### Copying the ESXi image files

```bash
mkdir /mnt/esxi7u2
```
```bash
mkdir /mnt/VMware-VMvisor-Installer-7.0U2a-17867351.x86_64.iso
```
```bash
mkdir /srv/tftpboot/esxi7u2
```
```bash
mkdir /srv/www/htdocs/images
```
```bash
mount /dev/sr0 /mnt/esxi7u2 (or) mount /dev/cdrom /mnt/esxi7u32
```
```bash
cp -rf /mnt/esxi7u2/* /srv/tftpboot/esxi7u2
```
```bash
cp -rf /mnt/VMware-VMvisor-Installer-7.0U2a-17867351.x86_64.iso /srv/www/htdocs/images
```
```bash
umount mnt/esxi7u2
```
>We need to remove references to the '/' path in the boot.cfg files
```bash
sed -i 's#/##g' /srv/tftpboot/esxi7u2/boot.cfg
```
```bash
sed -i 's#/##g' /srv/tftpboot/esxi7u2/efi/boot/boot.cfg
```

>Now adjust the configs of both boot.cfg files to the ones in the repo.

Now we create a mboot.efi file:
```bash
cp /srv/tftpboot/esxi7u2/efi/boot/bootx64.efi /srv/tftpboot/esxi7u2/mboot.efi
```
And we'll copy these two files to the root of the TFTP server
```bash
cp /srv/tftpboot/esxi7u2/boot.cfg /srv/tftpboot/boot.cfg
```
```bash 
cp /srv/tftpboot/esxi7u2/mboot.efi /srv/tftpboot/mboot.efi
```

### Legacy PXE Boot menu
```bash
mkdir /srv/tftpboot/pxelinux.cfg
```
```bash
touch /srv/tftpboot/pxelinux.cfg/default
```
```bash
cp /usr/share/syslinux/pxelinux.0 /srv/tftpboot
```
```bash
cp /usr/share/syslinux/menu.c32 /srv/tftpboot
```
>Edit the default config file to make a boot menu.

### Kickstart
Make a ks directory and config in the htdocs directory.
```bash
mkdir /srv/www/htdocs/ks
```
```bash
touch /srv/www/htdocs/ks/esxi.cgf
```
>And add the kickstart config from the repo.



