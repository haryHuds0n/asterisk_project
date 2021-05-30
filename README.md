### Before we begin
--------
Update system

```bash
sudo yum update
```
Disable SELinux and reboot machine

```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sudo systemctl reboot
```

### Configure firewall
--------

We need to add the SIP services to firewall configuation

```bash
sudo firewall-cmd --zone=public --permanent --add-service={sip,sips}
```

Add two ports for WebRTC

```bash
sudo firewall-cmd --zone=public --permanent --add-port=
8089/tcp
sudo firewall-cmd --zone=public --permanent --add-port=
8088/tcp
```

Add HTTP and HTTPS services

```bash
sudo firewall-cmd --zone=public --permanent --add-service={http,https}
```

### Install PJPROJECT
--------

Install build dependencies

```bash
sudo yum install epel-release gcc-c++ ncurses-devel libxml2-devel wget openssl-devel newt-devel kernel-devel-`uname -r` sqlite-devel libuuid-devel gtk2-devel jansson-devel binutils-devel bzip2 patch libedit libedit-devel
```

Make a directory for the build

```bash
mkdir ~/asterisk
```

Change to that directory and starting download PJPROJECT

```bash
wget https://www.pjsip.org/release/2.8/pjproject-2.8.tar.bz2
```

Extract it

```bash
tar -jxvf pjproject-2.8.tar.bz2
```

Change to extracted directory and run configure file with flags and options

```bash
./configure CFLAGS="-DNDEBUG -DPJ_HAS_IPV6=1" --prefix=/usr --libdir=/usr/lib64 --enable-shared --disable-video --disable-sound --disable-opencore-amr
```

Ensure that all dependencies are in place and build the plugin

```bash
make dep
make
```

Install the packages:

```bash
sudo make install
sudo ldconfig
```

### Install Asterisk
--------

Change back to directory that we create early and starting download asterisk

```bash
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-16-current.tar.gz
```

Unzip the file

```bash
tar -zxvf asterisk-16-current.tar.gz
```

Change to extracted directory

```bash
cd asterisk-16.1.1
```

### Configure and Build Asterisk
--------

Run `configure` file to prepare source code for compiling

```bash
./configure --libdir=/usr/lib64 --with-jansson-bundled
```
After configuation complete we can select more features we want to build

```bash
make menuselect
```

At this point we need to ensure following resources module is installed

```bash
res_srtp
res_cryto
```

After that we need to compile Asterisk with

```bash
make
```
When done we need to install Asterisk

```bash
sudo make install
```

We can install sample configuration files with

```bash
sudo make samples
```

### Test Connection

Start Asterisk

```bash
sudo systemctl start asterisk
```

To ensure that asterisk service starts even after a reboot, enable the service

```bash
sudo systemctl enable asterisk
```

To access Asterisk CLI

```bash
sudo asterisk -rvvvvvvv
``` 

