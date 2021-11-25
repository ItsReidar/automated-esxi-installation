# automated-esxi-installation
Automated installation of VMware ESXi 7.0 via a PXE server, with kickstart

Start with installing the TFTP server
```text
sudo zypper install tftp yast2-tftp-server
```
Next start the service and enable it (so it starts at boot)
```text
sudo systemctl start tftp.socket
sudo systemctl enable tftp.socket
```
Make sure to let UDP port 69 trough in the firewall or just disable the firewall on the machine (not recommended)

For the tftp config look in to the tftp file in the repo!


