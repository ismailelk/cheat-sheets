# Troubleshooting Network Services (EL7)

## Bottom-up approach

## Nework access layer
- Check the cables? Check if the devices are powered on, connection lights are on.
- VM (Virtualbox - Hyper-V)
  Check virtual network adapters and configuration
- `ip link`
- `ip route`
- `tracert`

ethernet adapter inschakelen `sudo dhclient`  
## Internet layer
1. LOCAL network configuration  
  - IP address: `ip a`  
  -  Default gateway: `ip r`  
  -  DNS service: `/etc/resolv.conf`
    - IP address?     
    - Correct subnet?  
    - DHCP of fixed?  
    - no APIPA?  
    - check config  
    `/etc/sysconfig/network-scripts/ifcfg-*`s
    Als een poort niet zichtbaar is, ook enable doen in configuration
    - /etc/sysconfig/network-scripts/ifcfg-eth1 'onboot' op yes plaatsen
    - check DHCP logs: `sudo journalctl -f`    
    - local configuration: `ip route`  
    - DNS Server: /etc/resolv.conf 'nameserver' option present? correct ip.

2. Routing with the LAN
    - ping between host
    - ping default Gateway and DNS
    - Query DNS (`dig , nslookup , getent`)  
      - host -> VM `ping 192.168.56.72`
      - VM -> host `ping 192.168.56.1`  
      - VM -> GW `ping 10.0.2.2`  
      - VM -> DNS `ping 10.0.2.3`  
      - DNS: `dig icanhazip.com`  
      - DNS: `nslookup icanhazip.com`  
      - DNS: `getent ahosts icanhazip.com`  
    Next step: Routing beyond GW -> Transport Layer  


## Transport layer
1. Service running? `sudo systemctl status SERVICE`
2. Correct port/interface? `sudo ss -tulpn`
3. Firewall setting? `sudo firewall-cmd --list-all`
  ### Firewall checks?
  - `sudo firewall-cmd --add-service=http`
  - `sudo firewall-cmd --add-service=https`
  - `sudo firewall-cmd --reload`
  - `sudo firewall-cmd --list-all`
  - `sudo firewall-cmd --add-interface=eth`
  - `sudo firewall-cmd --list-interfaces`
4. Is the service running?
  - `systemctl status httpd.service`
  - `systemctl start httpd`
  - `systemctl enable httpd`
  - `systemctl enable httpd --permanent`
5. Correct ports/interfaces?
  - `ss`
    - TCP service `sudo ss -tlnp`
    - UDP service `sudo ss -ulnp`
  - Correct port number?
    - see `/etc/services`
  - Correct interface?
    - Only loopback?
6. More firewall settings
  - `sudo firewall-cmd --list-all`
    - Is the service or port listed?
    - User `--add-service` if possible
    - Check with `--get-services`  
    OR USE, NOT BOTH
    - `--add-port`  
    -`--reload`  

## Application layer
- Check the logs: `journalctl`
  - Option 1: `journalctl -f -u httpd.service`
  - Option 2: `tail -f /var/log/httpd/error_log`
- Validate config file syntax?
  - for apache? `apachectl configtest`
- use client tools?
  - `curl , smbclient (samba), dig (dns)`
- Check the applications for errors? Specific manuals
## SELinux
- Settings
  - Booleans: `getsebool , setsebool`
  - Contexts, labels: `ls -Z, chcon , restorecon`
  - Policy modules: `sepolicy`
- Check file Context
  - Is the file context as expected?
    - `ls -Z /var/www/html`
  - Set file context to default value
    - `sudo restorecon -R /var/www/`
  - Set file context to specified value
    - `sudo chcon -t httpd_sys_content_t test.php`
- Check Booleans
  - `getsebool -a | grep http`
    - Know the relevant booleans (RedHat manuals)
    - Enable boolean:
      - `sudo setsebool -P httpd_can_network_connectdb on`
- Creating a Policy
   1. Allow Apache to run in "permissive" mode:   
    `$ sudo semange permissive -a httpd_t`
   2. Generate "Type Enforcement" file   
    `$ sudo audit2allow -a -m httpd-vboxsd > httpd-vboxsf.te`
   3. If necessary, edit the Policy  
    `$ sudo vi httpd-vboxsf.te`



Manuals  
[SELinux Users and Administrators's guide ](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/pdf/selinux_users_and_administrators_guide/Red_Hat_Enterprise_Linux-7-SELinux_Users_and_Administrators_Guide-en-US.pdf)  
[Networking Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/pdf/networking_guide/Red_Hat_Enterprise_Linux-7-Networking_Guide-en-US.pdf)  
[System Administrator's Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/pdf/system_administrators_guide/Red_Hat_Enterprise_Linux-7-System_Administrators_Guide-en-US.pdf)    

## Information

TCP/IP
|Layer| Protocols | Keywords
|Application| HTTP, DNS, SMB,FTP| |
|Transport | TCP, UDP | sockets, port numbers|
|Internet | IP, ICMP | routing, IP address |
|Network access | Ethernet | switch, MAC address |
|Physical | |cables|
