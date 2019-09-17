# Share Internet with Virtual Raspberry Pi using QEMU

1. Install **brctl**, **tunctl**, **libvirt**, **dnsmasq**, **qemu** and **kvm**
```
sudo apt-get install bridge-utils uml-utilities dnsmasq libvirt-bin qemu qemu-kvm
```

2. Make sure that DNS server and bridge automatically configured, for example:
```
$ ps -ef | grep dns
121       1377     1  0 Dec13 ?        00:00:00 /usr/sbin/dnsmasq -u libvirt-dnsmasq --strict-order --bind-interfaces --pid-file=/var/run/libvirt/network/default.pid --conf-file= --except-interface lo --listen-address 192.168.122.1 --dhcp-range 192.168.122.2,192.168.122.254 --dhcp-leasefile=/var/lib/libvirt/dnsmasq/default.leases --dhcp-lease-max=253 --dhcp-no-override
$
$ ifconfig virbr0
virbr0    Link encap:Ethernet  HWaddr 62:80:41:0f:0b:56  
          inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:619 errors:0 dropped:0 overruns:0 frame:0
          TX packets:685 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:44276 (44.2 KB)  TX bytes:51576 (51.5 KB)  
$
$ brctl show
bridge name bridge id  STP enabled interfaces 
virbr0  8000.000000000000 yes  
```

3. Bind eth0 to virbr0
```
sudo brctl addif virbr0 eth0
```
4. Create and send up a tap interface tap0
```
sudo tunctl -t tap0
sudo ifconfig tap0 up
```
5. Bind tap0 to virbr0
```
sudo brctl addif virbr0 tap0
```
6. Make sure that bindings are successful, for example:
```
$ brctl show
bridge name bridge id  STP enabled interfaces
virbr0  8000.1a9819c54ba8 yes       eth0
                                    tap0
```

7. Start QEMU with NIC
```
sudo qemu-system-arm \
-kernel kernel-qemu-4.19.50-buster \
-append "rw console=ttyAMA0 root=/dev/sda2 rootfstype=ext4 loglevel=8 rootwait fsck.repair=yes memtest=1" \
-dtb versatile-pb.dtb \
-drive file=2019-07-10-raspbian-buster-lite.img,format=raw \
-m 256 -M versatilepb -cpu arm1176 -no-reboot \
-net nic,macaddr=52:54:be:36:42:a9 \
-net tap,ifname=tap0,script=no,downscript=no \
-nographic
```
