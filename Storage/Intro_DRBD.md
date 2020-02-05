# Tìm hiểu DRBD

- [1. DRBD là gì?](#khainiem)
- [2. Cách DRBD hoạt động](#quytrinh)
- [3. Các chế độ đồng bộ của DRBD](#chedo)
  - [3.1 Protocol A](#A)
  - [3.2 Protocol B](#B)
  - [3.3 Protocol C](#C)
- [4. Cấu trúc của DRBD](#cautruc)
- [5. Tham khảo](#thamkhao)

## 1. DRBD là gì? 
<a name="khainiem"></a>

`DRBD` viết tắt của **Distributed Replicated Block Device**, là một tiện ích sử dụng để nâng cao tính sẵn sàng của hệ thống. Nó là được xây dựng trên nền ứng dụng mã nguồn mở để đơn giản hóa việc chia sẻ các dữ liệu trong hệ thống lưu trữ với nhau qua đường truyền mạng. Chúng ta có thể hiểu nôm na rằng đây là RAID-1 sử dụng các giao tiếp mạng để trao đổi dữ liệu cho nhau.

Về tổng quan, DRBD gồm 2 server cung cấp 2 tài nguyên lưu trữ độc lập, không liên quan gì với nhau. Trong một thời điểm, một server sẽ được cấu hình làm node `primary` đọc và ghi dữ liệu; node còn lại là `secondary` làm nhiệm vụ đồng bộ dữ liệu từ node primary để đảm bảo tính đồng nhất dữ liệu 2 node.

## 2. Cách hoạt động của DRBD 
<a name="quytrinh"></a>

Thường thì DRBD sẽ hoạt động theo cơ chế Active-Passive. Ở chế dộ này, node chính sẽ lắng nghe và thực hiện các yêu cầu đọc ghi từ phía user và động thời ghi trên node phụ. Node phụ sẽ được kích hoạt thành node chính sau khi phát hiện node chính down.
DRBD hỗ trợ hai kiểu ghi dữ liệu là fully synchronous và asynchronous.

- Fully synchronous (đồng bộ): Việc ghi được coi là hoàn tất khi tất cả các node đều ghi dữ liệu lên đĩa hoàn tất. Do nó nó phụ thuộc vào tốc độ ghi của disk trên tất cả các node và tốc độ mạng.

- Asynchronous (không đồng bộ): Việc ghi dữ liệu được coi là hoàn tất khi dữ liệu trên node chính hoàn thành. Không phụ thuộc vào việc đồng bộ 
![](http://i.imgur.com/szNSyVY.png)

DRBD cũng có thể hoạt động ở chế độ Active-Active. Ở chế độ này cả hai node đều có thể ghi dữ liệu và sau đó đồng bộ dữ liệu cho nhau.

## 3. Các chế độ đồng bộ của DRBD 
<a name="chedo"></a>

DRBD hỗ trợ 3 chế độ.

### 3.1. Protocol A 
<a name="A"></a>

Giao thức sao chép không đồng bộ. Quá trình ghi dữ liệu được coi là hoàn tất ngay sau khi node chính ghi hoàn tất hoàn tất. Đồng thời quá trình gửi và ghi dữ liệu trên các node khác cũng được thực hiện. Dữ liệu đồng bộ có thể hỏng nếu quá trình gửi dữ liệu bị dán đoạn.

### 3.2. Protocol B 
<a name ="B"></a>

Giao thức sao chép bán đồng bộ. Việc ghi được coi là hoàn thành ngay khi xảy ra việc ghi vào đĩa trên node chính và dữ liệu đã được gửi đến các node ngang hàng. Dữ liệu có thể bị hỏng nếu trong quá trình ghi vào disk bị gián đoạn

### 3.3. Protocol C
<a name ="C"></a>

Giao thức sao chép đồng bộ. Việc ghi được coi là hoàn tất khi việc ghi trên disk nội bộ và trên tất cả các node khác hoàn tất và được xác nhận. Tuy có tốc độ chậm nhưng giao thức này đảm bảo dữ liệu trên tất cả các node đều được đồng bộ và đầy đủ. Đây là giao thức được sử dụng phổ biến nhất.



## LAB Cài đặt và cấu hình cơ bản DRBD trên Centos 7

### Chuẩn bị.

- Hai server centos 7
- Hai thư mục trên mỗi server
- Cấu  hình hostname cho các server
- mở port 7788 trên các server.
- Đăng nhập với tài khoản root.
- Tạo phân vùng trên cả hai server.


Cụ thể:
**Note 1**
```
OS: CentOS 7 64 bit
Hostname: cen1
IP: 192.168.104.128
Gateway: 192.168.104.1
Network: 192.168.104.0/24
Partition: /dev/sda3
```
**Node 2**
```
OS: CentOS 7 64 bit
Hostname: cen2
IP: 192.168.104.129
Gateway: 192.168.104.1
Network: 192.168.104.0/24
Partition: /dev/sda3
```

### Mô hình

![](http://i.imgur.com/9ixEuBy.png)

### Các bước thực hiện:

#### Cài đặt DRBD

Cấu hình thêm host trên cả hai server:
```
cat << EOF >> /etc/hosts
[drbd]
192.168.104.128 cen1
192.168.104.129 cen2
EOF
```

Cài drbd trên cả hai server:
- Cấu hình repo và cài đặt module drbd
```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
yum install -y kmod-drbd84 drbd84-utils
```
- Bật module:
```
modprobe drbd
```
- Kiểm tra module đã hoạt động chưa.
```
lsmod | grep drbd

drbd                  405309  0
libcrc32c              12644  2 xfs,drbd
```


Tắt SELinux:
- Sửa trong file **/etc/selinux/config** dòng `SELINUX=enforcing` thành `SELINUX=disabled` để tắt SELinux mỗi lần khởi động. 
- Chạy lệnh sau để tạm thời tắt SELinux:
```
setenforce 0
```
Cấu hình firewall:
- Node1:
```
firewall-cmd --permanent --add-rich-rule='rule family="ipv4"  source address="192.168.104.129" port port="7789" protocol="tcp" accept'
firewall-cmd --reload
```

- Node2:
```
firewall-cmd --permanent --add-rich-rule='rule family="ipv4"  source address="192.168.104.128" port port="7789" protocol="tcp" accept'
firewall-cmd --reload
```


#### Cấu hình DRBD

File cấu hình chính của DRBD là **/etc/drbd.conf**. File này sẽ đọc các file **.res** trong thư mục **/etc/drbd.d/**

Tạo file cấu hình **/etc/drbd.d/testdata.res** trên cả hai server với nội dung sau:
```
resource testdata {
protocol C;
on cen1 {
    device /dev/drbd0;
    disk /mnt/testoncen1;
    address 192.168.104.128:7788;
    meta-disk internal;
    }
on cen2 {
    device /dev/drbd0;
    disk /mnt/testoncen2;
    address 192.168.104.129:7788;
    meta-disk internal;
    }
}
```
Giải thích:

- `resource testdata`: Tên của resource
- `protocol C`: Các resource được cấu hình để synchronous replication. 
- `cen1, cen2`: Danh sách các node và các tùy chọn bên trong
- `device /dev/drbd0`: Xác định thiết bị logic được DRBD sử dụng (Nên đặt giống nhau ở trên 2 server)
- `disk /dev/sdb`: Xác định thiết bị vật lý dùng để tạo ra thiết bị logic bên trên, và không nhất thiết phải cùng trên trên 2 server.
- `address 192.168.100.197:7788: Xác định địa chỉ IP và Port của mỗi server
- meta-disk internal: Cho phép sử dụng Meta-data mức nội bộ
Sao chép file cấu hình sang server 2:


#### Khởi động trên mỗi node
- Trên server 1:
```
[root@cen1 ~] drbdadm create-md testdata
```
- Trên server 2:
```
[root@cen2 ~] drbdadm create-md testdata
```
- Nếu output như sau là tạo thành công:
```
New drbd meta data block successfully created.
success
```
#### Bật DRBD daemon
Trên cả hai server chạy các lệnh sau để khởi đông và enable dịch vụ drdb:
```
systemctl start drbd 
systemctl enable drbd 
```

#### Kích hoạt trên node chính.

Cấu hình server cen1 làm node chính (primary) trong mô hình.

```
[root@cen1 ~] drbdadm primary testdata --force
```
Kiểm tra trạng thái:

```
[root@cen1 ~]# cat /proc/drbd
version: 8.4.7-1 (api:1/proto:86-101)
GIT-hash: 3a6a769340ef93b1ba2792c6461250790795db49 build by phil@Build64R7, 2016-01-12 14:29:40
 0: cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate C r-----
    ns:1048508 nr:0 dw:0 dr:1049236 al:8 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0
```

Hoặc dùng lệnh:

```
[root@cen1 ~]# drbd-overview
 0:testdata/0  Connected Primary/Secondary UpToDate/UpToDate
```

**Chú ý**: Chúng ta kiểm tra liên tục bằng lệnh trên và khi nào lệnh trả về kết quả tương tự hoặc có chứa nội dung `Connected Primary/Secondary` thì mới có thể chuyển sang bước 2.6.

#### Tạo mà mount file system DRBD

Tạo một file system và mount, ghi dữ liệu lên nó. Các bước thực hiện trên node chính - node mà bạn đã kích hoạt ở bước 2.5

```
[root@cen1 ~]# mkfs.ext3 /dev/drbd0
[root@cen1 ~]# mount /dev/drbd0 /mnt
[root@cen1 ~]# touch /mnt/testfile
[root@cen1 ~]# ll /mnt/
total 16
drwx------ 2 root root 1384 Oct  12 08:29 lost+found
-rw-r--r-- 1 root root    0 Oct  12 08:31 testfile
```


#### Test hoạt động replicate trên server 2

Chúng ta chuyển primary node sang cen2 để kiểm tra dữ liệu có được replicate

Unmount file system trên cen1

```
[root@cen1 ~]# umount /mnt
```

Chuyển sang chế độ `secondary node` cho cen1

```
[root@cen1 ~] drbdadm secondary testdata
```

Trên cen2, chúng ta kích hoạt chế độ primary

```
[root@cen2 ~] drbdadm primary testdata
```

Mount và kiểm tra dữ liệu bên trong

```
[root@cen2 ~]# mount /dev/drbd0 /mnt
[root@cen2 ~]# ll /mnt/
total 16
drwx------ 2 root root 1384 Oct  12 08:29 lost+found
-rw-r--r-- 1 root root    0 Oct  12 08:31 testfile
```

Kết quả trên cho ta thấy, dữ liệu đã được replicate sang cen2.










Nguồn tham khảo từ:
- https://github.com/hoangdh/ghichep-DRBD/blob/master/Cau-hinh-DRBD-co-ban-tren-CentOS.md
- https://docs.linbit.com/docs/users-guide-8.4
- https://github.com/hoangdh/ghichep-DRBD/blob/master/Cau-hinh-DRBD-co-ban-tren-CentOS.md
- https://www.tecmint.com/setup-drbd-storage-replication-on-centos-7/

