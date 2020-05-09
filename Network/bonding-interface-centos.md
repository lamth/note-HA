# Cấu hình bonding trên centos 7

## Khái niệm
#### Network bonding là gì? 
Hiểu đơn giản network bonding là cấu hình cho hai hay nhiều card mạng chạy song song để hỗ trợ, cân bằng tải và tăng khả năng chịu lỗi cho server. Nếu một trong các card trong bonding fail thì server vẫn có thể kết nối bình thường thông qua các card còn lại.


#### Các chế độ bonding
- **mode=0**: Balance Round Robin 
- **mode=1**: Active Backup
- **mode=2**: Balance XOR
- **mode=3**: Broadcast
- **mode=4**: 802.3ad
- **mode=5**: Balance TLB
- **mode=6**: Balance ALB


## Cấu hình

### Mô hình
- Môi trường thực hiện: Centos 7
- Network Interface:
  - `bond0`: `ens3` và `ens9` , network: 192.168.122.0/24
  - `bond1`: `ens10` và `ens11` , network: 192.168.123.0/24
- User: root
### Cấu hình thủ công bằng file config
**Bước 1: Bật module bonding**
Chạy lệnh sau:
```
modprobe bonding
```
**Bước 2: Tạo file cấu hình cho các interface bond**
Tạo file cấu hình cho interface `bond0` và đặt địa chỉ ip cho bond:
```
vi /etc/sysconfig/network-scripts/ifcfg-bond0
```
```conf
DEVICE=bond0
TYPE=Bond
NAME=bond0
BONDING_MASTER=yes
IPADDR=192.168.122.50
NETMASK=255.255.255.0
GATEWAY=192.168.122.1
DEFROUTE=yes
DNS1=8.8.8.8
USERCTL=no
BOOTPROTO=static
ONBOOT=yes
```
Tạo file cấu hình cho interface `bond1` và đặt địa chỉ Ip cho nó:
```
vi /etc/sysconfig/network-scripts/ifcfg-bond1
```
```conf
DEVICE=bond1
TYPE=Bond
NAME=bond0
BONDING_MASTER=yes
IPADDR=192.168.123.50
NETMASK=255.255.255.0
USERCTL=no
BOOTPROTO=static
ONBOOT=yes
```
**Bước 3: Cấu hình cho các Interface join interface bond**
Lưu ý: Nếu sử dụng ssh(hoặc một phương thức nào khác) để truy cập vào server theo địa chỉ ip của các interface chuẩn bị cấu hình thì kết nối sẽ bị mất.

- Cấu hình cho các interface thuộc `bond0`:
  - Tạo file backup cấu hình cũ của hai interface `ens3` và `ens9` nếu có:
  ```
  mv /etc/sysconfig/network-scripts/ifcfg-ens3 /etc/sysconfig/network-scripts/bak.ifcfg-ens3
  mv /etc/sysconfig/network-scripts/ifcfg-ens9 /etc/sysconfig/network-scripts/bak.ifcfg-ens9
  ```
  - Sửa lại file config `ens3` ở các dòng có sẵn hoặc thêm vào các dòng thiếu:
  ```
  vi /etc/sysconfig/network-scripts/ifcfg-ens3
  ```
  ```conf
  DEVICE=ens3
  BOOTPROTO=none
  ONBOOT=yes
  MASTER=bond0
  SLAVE=yes
  ```
  - Sửa lại file cấu hình `ens9`:
  ```
  vi /etc/sysconfig/network-scripts/ifcfg-ens9
  ```
  ```conf
  DEVICE=ens9
  BOOTPROTO=none
  ONBOOT=yes
  MASTER=bond0
  SLAVE=yes
  ```
- Cấu hình cho các interface thuộc `bond1`:
  - Tạo file backup cấu hình cũ của hai interface `ens10` và `ens11` nếu có:
  ```
  mv /etc/sysconfig/network-scripts/ifcfg-ens11 /etc/sysconfig/network-scripts/bak.ifcfg-ens11
  mv /etc/sysconfig/network-scripts/ifcfg-ens10 /etc/sysconfig/network-scripts/bak.ifcfg-ens10
  ```
  - Tạo file cấu hình mới cho interface `ens10`:
  ```
  vi /etc/sysconfig/network-scripts/ifcfg-ens10
  ```
  ```conf
  DEVICE=ens10
  BOOTPROTO=none
  ONBOOT=yes
  MASTER=bond0
  SLAVE=yes
  ```
  - Tạo mới file cấu hình `ens11`:
  ```
  vi /etc/sysconfig/network-scripts/ifcfg-ens11
  ```
  ```conf
  DEVICE=ens11
  BOOTPROTO=none
  ONBOOT=yes
  MASTER=bond0
  SLAVE=yes
  ```

- Sau khi tạo file cấu hình cho hai bond(bond0 và bond1) và cấu hình các interfaces làm slave của các bond, chạy lệnh sau để hệ thống cập nhật lại cấu hình:
```
systemctl restart network
```

- Nếu kết nối đến server theo địa chỉ cũ thì sẽ bị mất kết nối, thiết lập lại bằng cách kết nối đến địa chỉ IP mới trên interface bond.

- Kiểm tra cấu hình bằng lệnh `ip a` thấy các interface `ens3`, `ens9`, `ens10`, `ens11` đã up và ở chế độ **slave**, `bond0` và `bond1` đã nhận địa chỉ ip:
```
[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 52:54:00:c5:df:bf brd ff:ff:ff:ff:ff:ff
3: ens9: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 52:54:00:c5:df:bf brd ff:ff:ff:ff:ff:ff
4: ens10: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond1 state UP group default qlen 1000
    link/ether 52:54:00:35:b9:71 brd ff:ff:ff:ff:ff:ff
5: ens11: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master bond1 state UP group default qlen 1000
    link/ether 52:54:00:35:b9:71 brd ff:ff:ff:ff:ff:ff
7: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 52:54:00:c5:df:bf brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.50/24 brd 192.168.122.255 scope global noprefixroute bond0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fec5:dfbf/64 scope link tentative dadfailed 
       valid_lft forever preferred_lft forever
8: bond1: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 52:54:00:35:b9:71 brd ff:ff:ff:ff:ff:ff
    inet 192.168.123.50/24 brd 192.168.123.255 scope global noprefixroute bond1
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe35:b971/64 scope link tentative dadfailed 
       valid_lft forever preferred_lft forever
```
- Trạng thái chi tiết của từng bond có thể kiểm tra ở file `/proc/net/bonding/<tên-interface-bond>`. ví dụ:
```yml
[root@localhost ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: load balancing (round-robin)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: ens3
MII Status: up
Speed: 100 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 52:54:00:c5:df:bf
Slave queue ID: 0

Slave Interface: ens9
MII Status: up
Speed: 100 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 52:54:00:f2:f1:f3
Slave queue ID: 0
```

Ngoài ra có thể cấu hình thêm chế độ bonding bằng tùy chọn `BONDING_OPTS` trong file cấu hình, mặc định nếu không có tùy chọn này thì chế độ bonding là LB round-robin

## Nguồn:
- https://secure.vinahost.vn/ac/knowledgebase/247/Hng-dn-cu-hinh-bonding-2-card-mng-RHELorCentOS.html
