Pour acceder au réseau 248 par ssh, doit:
  - definir une gateway sur 159.31.247  : 159.31.247.1
  - pas de gateway sur 159.31.248
  - dns: 159.31.10.2  /10.4

Pour permetre un ssh sur 248, doit utiliser les policy-based routing" (routage basé sur une politique)
 -  rajouter dans /etc/iproute2/rt_tables 
200   custom_table
 - ip rule add from 159.31.248.1XX table custom_table
 - ip route add default via 159.31.248.1 dev eno4 table custom_table

 Pour que cela soit permanent, rajouter /etc/systemd/system/custom-routing.service
[Unit]
Description=Custom Routing Rules
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/bin/env /root/custom-routing-script.sh
RemainAfterExit=true

[Install]
WantedBy=multi-user.target

et custom-routing-script.sh

#/bin/bash

ip rule add from 159.31.248.11 table custom_table
ip route add default via 159.31.248.1 dev enoXXXX table custom_table

Puis: 
chmod +x /root/custom-routing-script.sh
systemctl enable custom-routing
systemctl start custom-routing

Concernant les IP:

VM sur 159.31.247

lame1 idrac: 159.31.248.10
lame1 hyperviseur: 159.31.248.11


lame2 idrac: 159.31.248.12
lame2 hyperviseur: 159.31.248.13


Pour les differents mode de bridges de la vm:
https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking#
https://linuxconfig.org/how-to-use-bridged-networking-with-libvirt-and-kvm

Le plus simple est de definir un reseau de type macvtap comme dans 
https://blog.scottlowe.org/2016/02/09/using-kvm-libvirt-macvtap-interfaces/
puis de le connecter via un reseau virtuel. 


En cas de pb avec une VM, la supprimer:
virsh undefine ceph-1 --remove-all-storage --wipe-storage --snapshots-metadata --managed-save

Doit avoir 
export VIRSH_DEFAULT_CONNECT_URI=qemu:///system
export LIBVIRT_DEFAULT_URI=qemu:///system

Car sinon execute virsh en mode session, ce qui donne n'importe quoi.



