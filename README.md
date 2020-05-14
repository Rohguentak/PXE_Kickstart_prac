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
          # wget http://mirror.navercorp.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-2003.iso  //iso 파일 
	  # mkdir /media/iso
	  # mount -o loop /home/roh/CentOS-7-x86_64-Minimal-2003.iso /media/iso/
	  mount: /dev/loop0 is write-protected, mounting read-only
	  # cp /media/iso/isolinux/vmlinuz /tftpboot/CentOS7/
	  # cp /media/iso/isolinux/initrd.img /tftpboot/CentOS7/
	  # cp /root/anaconda-ks.cfg /tftpboot/ks/ks.cfg
	  # vi /tftpboot/ks/ks.cfg 
		#version=DEVEL
		# System authorization information
		auth --enableshadow --passalgo=sha512
		# Use CDROM installation media
		install
		# Use graphical install
		text

		nfs --server=172.28.128.10 --dir="/iso"
		# Run the Setup Agent on first boot
		firstboot --enable
		ignoredisk --only-use=sda
		# Keyboard layouts
		keyboard --vckeymap=kr --xlayouts='kr','us'
		# System language
		lang ko_KR.UTF-8

		# Network information
		network  --bootproto=dhcp --device=enp0s3 --ipv6=auto --activate
		network  --bootproto=dhcp --device=enp0s8 --onboot=on --ipv6=auto
		network  --hostname=localhost.localdomain

		# Root password
		rootpw --iscrypted $6$k/U0AP/NPLjFvAs2$vTkY92fFB0/.PbuqbsjJlYyL3bPASiOKlkktXEW8kqZCHnIGUmq89w0a8oSrNGRX.3QNxUBiAzrP.z8pIKzWb0
		# System services
		services --enabled="chronyd"
		# System timezone
		timezone Asia/Seoul --isUtc
		user --groups=wheel --name=roh --				password=$6$dubHTrc/vCo1zgLG$o3leCuSjV2OG98uoZa6us.4MmKj3hcAMZx4qkhR7CNfrftfepB5X5VmQ.vwWpjf3Cc3u2xv5r..NzPdpuYEjf. --iscrypted --gecos="roh"
		# System bootloader configuration
		bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
		autopart --type=lvm
		# Partition clearing information
		clearpart --none --initlabel

		%packages
		@^minimal
		@core
		chrony
		kexec-tools

		%end

		%addon com_redhat_kdump --enable --reserve-mb='auto'

		%end

		%anaconda
		pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
		pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
		pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
		%end
	# chmod -R 777 /tftpboot/
	# systemctl stop firewalld
	# systemctl disable firewalld
	Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
	Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
	# vi /etc/sysconfig/selinux 
		# This file controls the state of SELinux on the system.
		# SELINUX= can take one of these three values:
		#     enforcing - SELinux security policy is enforced.
		#     permissive - SELinux prints warnings instead of enforcing.
		#     disabled - No SELinux policy is loaded.
		SELINUX=disabled
		# SELINUXTYPE= can take one of three values:
		#     targeted - Targeted processes are protected,
		#     minimum - Modification of targeted policy. Only selected processes are protected. 
		#     mls - Multi Level Security protection.
		SELINUXTYPE=targeted 
	# vi /etc/dhcp/dhcpd.conf 
		#
		# DHCP Server Configuration file.
		#   see /usr/share/doc/dhcp*/dhcpd.conf.example
		#   see dhcpd.conf(5) man page
		#
		allow booting;		
		allow bootp;
		default-lease-time 600;
		max-lease-time 7200;
		option domain-name-servers 192.168.21.1;
		ddns-update-style none;
		next-server 172.28.128.10;
		filename "pxelinux.0";
		subnet 172.28.128.0 netmask 255.255.255.0 {
		option routers 172.28.128.1;
		range 172.28.128.11 172.28.128.20;
		}
	# cp -r /media/iso/* /iso
	# vi /etc/exports
	# cat /etc/exports
	/tftpboot/ks *(ro)
	/iso *(ro)
	# systemctl restart dhcpd
	# systemctl restart nfs
	# systemctl restart xinetd
	# systemctl restart tftp


		
			
#client는 네트워크로 부팅되도록 설정 하면 os 설치됨
