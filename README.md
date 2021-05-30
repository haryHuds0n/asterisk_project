This repo create for purpose take note for pjsip configuation with sipml5

###Before we begin
--------
Update system
```
sudo yum update
```
Disable SELinux and reboot machine
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

sudo systemctl reboot
```

### Configure firewall
--------
We need to add the SIP services to firewall configuation
```
sudo firewall-cmd --zone=public --permanent --add-service={sip,sips}
```

Add two ports for WebRTC
```
sudo firewall-cmd --zone=public --permanent --add-port=
8089/tcp
sudo firewall-cmd --zone=public --permanent --add-port=
8088/tcp
```

Add HTTP and HTTPS services
```
sudo firewall-cmd --zone=public --permanent --add-service={http,https}
```

### Install PJPROJECT
--------
Install build dependencies
```
sudo yum install epel-release gcc-c++ ncurses-devel libxml2-devel wget openssl-devel newt-devel kernel-devel-`uname -r` sqlite-devel libuuid-devel gtk2-devel jansson-devel binutils-devel bzip2 patch libedit libedit-devel
```
Make a directory for the build
```
mkdir ~/asterisk
```
Change to that directory and starting download PJPROJECT
```
wget https://www.pjsip.org/release/2.8/pjproject-2.8.tar.bz2
```
Extract it
```
tar -jxvf pjproject-2.8.tar.bz2
```
Change to extracted directory and run configure file with flags and options
```
./configure CFLAGS="-DNDEBUG -DPJ_HAS_IPV6=1" --prefix=/usr --libdir=/usr/lib64 --enable-shared --disable-video --disable-sound --disable-opencore-amr
```
Ensure that all dependencies are in place and build the plugin
```
make dep
make
```
Install the packages:
```
sudo make install
sudo ldconfig
```

### Install Asterisk
--------
Change back to directory that we create early and starting download asterisk
```
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-16-current.tar.gz
```
Unzip the file
```
tar -zxvf asterisk-16-current.tar.gz
```
Change to extracted directory
```
cd asterisk-16.1.1
```

### Configure and Build Asterisk
--------

Run `configure` file to prepare source code for compiling
```
./configure --libdir=/usr/lib64 --with-jansson-bundled
```
After configuation complete we can select more features we want to build
```
make menuselect
```

At this point we need to ensure following resources module is installed
```
res_srtp
res_cryto
```
After that we need to compile Asterisk with
```
make
```
When done we need to install Asterisk
```
sudo make install
```
We can install sample configuration files with
```
sudo make samples
```

### Test Connection

Start Asterisk
```
sudo systemctl start asterisk
```

To ensure that asterisk service starts even after a reboot, enable the service
```
sudo systemctl enable asterisk
```

To access Asterisk CLI
```
sudo asterisk -rvvvvvvv
``` 