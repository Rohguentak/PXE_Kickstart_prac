# PXE_Kickstart_prac
--------------------
# 실습환경
          VM : virtualbox 5.2.26
          OS : CentOS 7.8
          vm spec: 메모리 4096MB 가상HDD 20GB
          server 네트워크 : adapter1 - NAT adapter2 - Hostonly
         
         
         
# pxe 설정 및 kickstart 실습
          #yum update -y
          #yum groupinstall -y "GNOME Desktop" "Graphical Administration Tools"
          #ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target   //GUI설정
          #reboot
          #ip addr show
          1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
              link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
              inet 127.0.0.1/8 scope host lo
                 valid_lft forever preferred_lft forever
              inet :: 1/128 scope host
                 valid_lft forever preferred_lft forever
          2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
              link/ether 08:00:27:34:88:e2 brd ff:ff:ff:ff:ff:ff
              inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic enp0s3
                 valid_lft forever preferred_lft 86058sec
              inet fe80::e94a:c781:2840:a4b/64 scope link noprefixroute
                 valid_lft forever preferred_lft forever
          3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
              link/ether 08:00:27:77:ea:af brd ff:ff:ff:ff:ff:ff
          #vi /etc/sysconfig/network-scripts/ifcfg-enp0s8
          TYPE=Ethernet
          PROXY_METHOD=none
          BROWSER_ONLY=no
          BOOTPROTO=static
          IPADDR=172.28.128.10
          NETMASK=255.255.255.0
          GATEWAY=172.28.128.1
          DEFROUTE=yes
          IPV4_FAILURE_FATAL=no
          IPV6INIT=yes
          IPV6_AUTOCONF=yes
          IPV6_DEFROUTE=yes
          IPV6_FAILURE_FATAL=no
          IPV6_ADDR_GEN_MODE=stable-privacy
          NAME=enp0s8
          UUID=6bbc2ae4-f822-4d90-9328-b07c9ee6e765
          DEVICE=enp0s8
          ONBOOT=yes
          #systemctl restart network
          1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
              inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
              inet6 ::1/128 scope host 
          valid_lft forever preferred_lft forever
          2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
          link/ether 08:00:27:34:88:e2 brd ff:ff:ff:ff:ff:ff
              inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic enp0s3
          valid_lft 86389sec preferred_lft 86389sec
              inet6 fe80::e94a:c781:2840:a4b/64 scope link noprefixroute 
          valid_lft forever preferred_lft forever
          3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
          link/ether 08:00:27:77:ea:af brd ff:ff:ff:ff:ff:ff
              inet 172.28.128.10/24 brd 172.28.128.255 scope global noprefixroute enp0s8
          valid_lft forever preferred_lft forever
              inet6 fe80::9f16:5a10:11ff:589b/64 scope link noprefixroute 
          valid_lft forever preferred_lft forever
          
          
          
          # yum install -y tftp tftp-server dhcp xinetd syslinux 
          # vi /etc/xinetd.d/tftp 
          # default: off
          description: The tftp server serves files using the trivial file transfer \
          protocol.  The tftp protocol is often used to boot diskless \
          workstations, download configuration files to network-aware printers, \
          and to start the installation process for some operating systems.
          service tftp
          {
	            socket_type		= dgram
	            protocol		  = udp
	            wait			    = yes
	            user			    = root
	            server			  = /usr/sbin/in.tftpd
	            server_args		= -s /tftpboot
	            disable			  = no
	            per_source		= 11
	            cps			      = 100 2
	            flags			     = IPv4
          }
          # mkdir /tftpboot
          # cp /usr/share/syslinux/menu.c32 /tftpboot/
          # cp /usr/share/syslinux/pxelinux.0 /tftpboot/
          # mkdir /tftpboot/CentOS7
          # mkdir /tftpboot/ks
          # mkdir /tftpboot/pxelinux.cfg
          # vi /tftpboot/pxelinux.cfg/default
          default menu.c32
          timeout 100

          menu title ### OS Installer Boot Menu ###

          LABEL CentOS7
	            kernel CentOS7/vmlinuz
	            append ksdevice=link load_ramdisk=1 initrd=CentOS7/initrd.img unsupported_hardware text network ks=nfs:172.28.128.10:/tftpboot/ks/ks.cfg text
          #wget http://mirror.navercorp.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-2003.iso  //iso 파일 

