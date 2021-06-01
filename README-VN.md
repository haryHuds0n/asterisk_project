##### TRƯỚC KHI BẮT ĐẦU
---
Update hệ thống

```bash
sudo yum update
```

Tắt chức năng SELinux
```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sudo systemctl reboot
```

##### CẤU HÌNH TƯỜNG LỬA
---
Thêm dịch vụ SIP, HTTP và HTTPS vào phần cấu hình tường lửa

```bash
sudo firewall-cmd --zone=public --permanent --add-service={sip,sips,http,https}
```
Thêm hai cổng sử dụng cho WebRTC vào cấu hình tường lửa

```bash
sudo firewall-cmd --zone=public --permanent --add-port=
8089/tcp
sudo firewall-cmd --zone=public --permanent --add-port=
8088/tcp
```

##### INSTALL PJPROJECT
---
Install các dependencies cho việc build Asterisk

```bash
sudo yum install epel-release gcc-c++ ncurses-devel libxml2-devel wget openssl-devel newt-devel kernel-devel-`uname -r` sqlite-devel libuuid-devel gtk2-devel jansson-devel binutils-devel bzip2 patch libedit libedit-devel
```
Tạo thư mục Asterisk

```bash
mkdir ~/asterisk
```
Di chuyển vào vừa tạo và thực hiện download PJPROJECT 

```bash
wget https://www.pjsip.org/release/2.8/pjproject-2.8.tar.bz2
```

Giải nén file vừa download về

```bash
tar -jxvf pjproject-2.8.tar.bz2
```

Di chuyển vào thư mục vừa giải nén và chạy file `configure` với các option đi kèm

```bash
cd pjproject-2.8
./configure CFLAGS="-DNDEBUG -DPJ_HAS_IPV6=1" --prefix=/usr --libdir=/usr/lib64 --enable-shared --disable-video --disable-sound --disable-opencore-amr
```

Build các plugin

```bash
make dep
make
```

Install các packages đi kèm

```bash
sudo make install
sudo ldconfig
```

##### CÀI ĐẶT ASTERISK
---

Di chuyển trở về thư mục `asterisk` đã tạo trước đó và thực hiện download Asterisk

```bash
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-16-current.tar.gz
```

Giải nén file vừa download về

```bash
tar -zxvf asterisk-16-current.tar.gz
```

Di chuyển vào thư mục vừa giải nén

```bash
cd asterisk-16.1.1
```

##### CẤU HÌNH VÀ XÂY DỰNG ASTERISK
---

Chạy file `configure` để chuẩn bị source code cho việc compile

```bash
./configure --libdir=/usr/lib64 --with-jansson-bundled
```

Sau khi quá trình `confiure` hoàn thành ta có thể chọn thêm các feature muốn build thêm với `menuselect`

```bash
make menuselect
```

Lúc này ta phải bảo đảm bảo rằng hai mục này đã được chọn trong `resource modules`

```bash
res_srtp
res_cryto
```

Sau khi chắc chắn ta có thể thực hiện compile Asteisk

```bash
make
```

Khi compile xong ta thực hiện cài đặt Asterisk

```bash
sudo make install
```

Ta có thể tạo ra các file cấu hình mẫu với command sau

```bash
sudo make samples
```

##### KIỂM TRA KẾT NỐI

Khởi động Asterisk

```bash
sudo systemctl start asterisk
```

Để bảo đảm Asterisk sẽ chạy khi khỏi động ta phải `enable` nó lên

```bash
sudo systemctl enable asterisk
```

Truy cập giao diện dòng lệnh của Asterisk

```bash
sudo asterisk -rvvvvvvv
``` 


##### CẤU HÌNH ASTERISK CHO WEBRTC CLIENTS

Để cấu hình Asterisk cho WebRTC Clients ta phải có `certificate`, hướng dẫn dưới dây sẽ giúp tạo self-sign certificates cho Asterisk, để tạo certificates ta sử dụng command trong thư mục của Asterisk mà ta đã download trước đó

```bash
sudo contrib/scripts/ast_tls_cert -C <IP-Address-of-Asterisk-Server> -O "My Organization" -b 2048 -d /etc/asterisk/keys
```

Thay thế `IP-Address-of-Asterisk-Server` với địa chỉ IP của Asterisk Server. Sau khi `Enter` ta sẽ phải nhập password một vài lần cho certificate
Sau khi xong, ta sẽ sử dụng `asterisk.crt` và `asterisk.key` để cấu hình cho HTTP và HTTPS

##### CẤU HÌNH ASTERISK
###### CẤU HÌNH ASTERISK BUILT-IN HTTP SERVER 

Cấu hình `/etc/asterisk/http.conf` 

```bash
[general]
enabled=yes
bindaddr=0.0.0.0
bindport=8088
tlsenable=yes
tlsbindaddr=0.0.0.0:8089
tlscertfile=/etc/asterisk/keys/asterisk.crt
tlsprivatekey=/etc/asterisk/keys/asterisk.key
```

Khởi động lại và kiểm tra TLS server đã chạy với Asteisk CLI command

```bash
http show status
```
###### CẤU HÌNH PJSIP
###### PJSIP WSS Transport

Cấu hình `/etc/asterisk/pjsip.conf`

```bash
[transport-wss]
type=transport
protocol=wss
bind=0.0.0.0
```
###### PJSIP Endpoint, AOR và Auth

```bash
[User1]
type=aor
max_contacts=5
remove_existing=yes

[User1]
type=auth
auth_type=userpass
username=User1
password=User1 

[User1]
type=endpoint
aors=User1
auth=User1
dtls_auto_generate_cert=yes
webrtc=yes
context=default
disallow=all
allow=opus,ulaw
```

##### WEBRTC SỬ DỤNG SIPML5

##### Cấu hình SIPML5

Truy cập [SIPML5](https://www.doubango.org/sipml5/) để cấu hình SIPML5 Client
Click vào "Enjoy our live demo" và cấu hình như sau 

![image](images/registration_box.png)

`Displayname:` Là tên hiển thị khi thực hiện cuộc gọi
`Private Identify:` Đây là tên đã được set trong `pjsip.conf`
`Public Identify:` Nhập theo format `sip:<Private Identify>@<IP-Address-Asterisk-Server`

Tiếp theo, click `expert mode` và cấu hình như sau

![image](images/expert_settings.png)

Cuối cùng, để có thể login được ta phải truy cập vào  `https://<IP-Asterisk-Server>:8089/ws` để chấp nhận certificate mà ta đã tạo trước đó. Sau khi `Accept` trang web sẽ giống như sau

![image](images/wss.png)

Sau khi hoàn thành bước trên ta có thể click `Login`, nếu xuất hiện `connected`
là đã login thành công

Để có thể thực hiện một cuộc gọi đơn giản ta cấu hình `extensions.conf` như sau

```bash
[default]

exten=>6001,1,Dial(PJSIP/User1,20)
exten=>6002,1,Dial(PJSIP/User2,20)
```
6001, 6002: Là ext để nhập vào thực hiện cuộc gọi
PJSIP: Thư viện được sử dụng
User1, User2: Là hai User đã được set trong `pjsip.conf`
##### THỰC HIỆN CUỘC GỌI
Để thực hiện được cuộc gọi ta login thêm một User tương tự ở một trình duyệt web khác
Trong box Call Control của `User1` ta nhập 6002 sau đó nhấn nút `Call`

![image](images/call-popup.png)

Click vào "Audio". Khi đó trình duyệt sẽ yêu cầu quyền cho phép truy cập vào microphone của máy

![image](images/allow_micro.png)

Click chọn "Allow."
Sau khi click `Allow` ta sẽ thấy cuộc gọi đang trong quá trình xử lý

![image](images/in_progress.png)


Khi bên `User2` kia nhận cuộc gọi, cuộc gọi sẽ được hiển thị trạng thái là `In Call`

![image](images/in_call.png)

Tới đây là ta đã thực hiện được một cuộc gọi bằng WebRTC thông qua Asterisk
Link tham khảo tài liệu:

[Configuring Asterisk for WebRTC Clients - Asterisk Project - Asterisk Project Wiki](https://wiki.asterisk.org/wiki/display/AST/Configuring+Asterisk+for+WebRTC+Clients)
[Stack Overflow - Where Developers Learn, Share, & Build Careers](https://stackoverflow.com/)
