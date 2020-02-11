# Đồng bộ dữ liệu bằng lsyncd.

Triển khai lab với mô hình 3 node master replicate dữ liệu cho nhau.

#### Mô hình

![](https://i.imgur.com/HjblJlk.png)

Danh sách các node:
- Node1:
  - OS: Centos 7
  - IP: 192.168.104.138/24
  - Folder: /root/test1
- Node2:
  - OS: Centos 7
  - IP: 192.168.104.139/24
  - Folder: /root/test2
- Node3:
  - OS: Centos 7
  - IP: 192.168.104.132/24
  - Folder: /root/test3
#### B1.Cài đặt:

Trên tất cả các node:
```
yum install -y epel-release
yum -y install lua lua-devel pkgconfig gcc asciidoc lsyncd
```

#### B2:
Cấu hình gen ssh key trên tất cả server và copy đến file authorized_keys trên tất cả server.

Đảm bảo tất cả các node đều có thể kết nối ssh với nhau sử dụng ssh key

#### Cấu hình trên node 1
Tạo hoặc sửa file cấu hình /etc/lsyncd.conf thành:

```lua
settings {
insist = true,
logfile = "/var/log/lsyncd/lsyncd.log",
statusFile = "/var/log/lsyncd/lsyncd.status",
statusInterval = 1
}
sync{
default.rsyncssh,
source="/root/test1",
host="192.168.104.139",
targetdir="/root/test2",
rsync = {
archive = true, 
perms = true, 
owner = true, 
_extra = {"-a"}, 
},
delay = 5,
maxProcesses = 1
}
sync{
default.rsyncssh,
source="/root/test1",
host="192.168.104.132",
targetdir="/root/test3",
rsync = {
archive = true, 
perms = true, 
owner = true, 
_extra = {"-a"}, 
},
delay = 5,
maxProcesses = 1
}
```

Tạo thư mục và file chứa log
```
mkdir -p /var/log/lsyncd
touch /var/log/lsyncd/lsyncd.log
touch /var/log/lsyncd/lsyncd.status
```
Khởi động dịch vụ:
```
systemctl restart lsyncd
systemctl enable lsyncd
```


#### Cấu hình trên node 2 và node 3
Cấu hình hai node còn lại giống node1. Trong file cấu hình `/etc/lsyncd.conf`, sửa phần source thành folder muốn sao chép, host và target đến địa chỉ và folder của các node còn lại trong mô hình.


#### Kiểm tra việc sao chép dữ liệu.

Tạo file trong thư mục /root/test1 trên node1:

![](https://i.imgur.com/Jqcm5Dh.png)

Kiểm tra các thư mục trên hai node còn lại:

![](https://i.imgur.com/IQPQU5t.png)

Trên node hai, xóa một file1 và kiểm tra trên các node còn lại thấy file cũng đã bị xóa:

![](https://i.imgur.com/2WKroxY.png)


Done.