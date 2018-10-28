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
    - DNS Server: `/etc/resolv.conf` 'nameserver' option present? correct ip.

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

|Layer    |Protocols | Keywords|
|-------|------------|--------|
|Application| HTTP, DNS, SMB,FTP|   |  
|Transport | TCP, UDP | sockets, port numbers|  
|Internet | IP, ICMP | routing, IP address |  
|Network access | Ethernet | switch, MAC address |  
|Physical | |cables|  

## Keyboard Settings

Setting the systeminput to azerty:
```
sudo localectl set-keymap be
sudo localectl set-x11-keymap be
sudo loadkeys fr
```

# Report

### Phase 1: Fysieke en Netwerktoegangslaag
- Staan alle aparaten aan?
- Bekabeling?
In VirtualBox, go to the VM settings, Network, select the active interfaces, click “Advanced” and make sure the checkbox “Cable connected” is checked.
- Adapters? Worden de verwachte soort netwerkadapters gebruikt?
- controleer of de LINK UP is, dit met het commando `ip link`

### Phase 2: Internetlaag
 #### Netwerk instellingen
 1. De netwerkinterface heeft een IP? Default Gateway? DNS Server?
 2. IP address: statisch of DHCP? /etc/sysconfig/network-scripts/ifcfd-IFACE
      - indien dynamisch "BOOTPROTO : dhcp"
      - indien statisch "BOOTPROTO : none" , "IPADDR= 192.168.56.24", "NETMASK = 255.255.255.0"
 3. Check actual value? `ip a`
 4. check DW : `ip r`
 5. check DNS `cat /etc/resolve.conf`
 6. pingen van andere hosts in je lan bij Windows wordt ICMP verkeer soms geblokeerd met die commando kan je dit oplossen
 `Get-NetFirewallRule -DisplayName "*Echo Request*" | Set-NetFirewallRule -enabled true`
 7. check DNS name resolution? `dig www.hogent.be +short` and `nslookup www.hogent.be`
 8. Query Specific DNS `dig www.hogent.be @8.8.8.8 +short` and `nslookup www.hogent.be 8.8.8.8`
 9. DNS if dig and nslookup are not avaiable  `getent ahosts www.google.com`

### Phase 3: Transportlaag
#### Service and port
1. Check if the service is running (ex. nginx)   
```
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
```
2. What port is the service using?  
`sudo ss -tlnp `  
  (list TCP (-t) server (-l) port numbers (-n) with the process behind them (-p). The -p option requires root, hence the sudo)  
  - Output is afhankelijk van wat je ingesteld hebt standaard http (80) en https (443).
   Check `/etc/services` voor de precieze poort van je service
  - Is de service ook aan het luisteren naar externe interfaces? Standaard staan de netwerkinstellingen van een service op luisteren naar loopback interface.

#### Firewall setting
1. Does the firewall allow traffico on the service?  
  `sudo firewall-cmd --list-all`  
2. Check if the service is listed?  
  `firewall-cmd --get-services` (grep kan helpen indien het er veel zijn)
### Phase 4: Applicatielaag
1. Check the logs (maybe you can find an indication) put a tail on the file  
    `journalctl` -> `sudo journalctl -f httpd.service`  
    `sudo tail -f /cat/log/httpd/error_log`  
2. Check config files  
  `httpd: apachectl configtest`  
  after making some changes -> `sudo systemctk restart httpd.service `
3. Check availability  
  - Do a port scan from another host on the lan  
    `sudo nmap -sS -p 80,443 HOST`
  - Use a test tool  
    `wget http://HOST/ , wget https://HOST/`  
    `curl http://HOST/ , curl https://HOST/`  
