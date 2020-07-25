# Tìm hiểu đồng bộ database của MariaDB với Galera.

## Galera Cluster

- MariaDB Galera Cluster là giải pháp đồng bộ multi-master cluster cho MariaDB database. Tức là các node trong cụm hoạt động ngang hàng, đồng bộ qua lại với nhau và tất cả các node đều có thể thực hiện nhiệm vụ như nhau(như đọc và ghi dữ liệu trên database).
- Các chức năng chính của Galera:
  - Giải pháp đồng bộ multi-master hoàn chỉn, nên cho phép đọc ghi trên bất kì node nào.
  - Đồng bộ giữa tất cả các node tham gia
  - Không cần các cơ chế failover vì node nào cũng là master rồi
  - Tự động kiểm soát thành viên , tham gia và rời khởi cụm
- Hạn chế của Galera:
  - Không tăng dung lượng lưu trữ: Vì trên tất cả các node dữ liệu đều giống nhau, đều tốn một lượng lưu trữ giống nhau nên Dung lượng lưu trữ của cả cụm phụ thuộc vào dung lượng lưu trữ từng node trong cụm.
  - Chỉ hỗ trợ innodb và không hỗ trợ MyISAM, chuyển đổi một database sử dụng các myisam table sang innodb để sử dụng galera cluster sẽ khó khăn.
- Một số yêu cầu khi sử dụng Galera:
  - Chỉ hỗ trợ innodb. Đây là yêu cầu thiết kế cơ sở dữ liệu.
  - Yêu cầu các node có cấu hình tương đương nhau và tổng số node phải là lẻ. Tối thiểu số node phải là 3. Nếu muốn biết tại sao lại là số lẻ thì đọc thêm về khái niệm quorum trong thiết kế High Availibility
  - flush privileges command không được replicate
  - Query log phải đổ vào file, không thể đổ vào table được.    

- Các thành phần trong Galera:
  - Database management System: Là các database server như MySQL, MariaDB,..
  - wsrepAPI: Cung cấp giao diện cho database server và nó cung cấp việc đồng bộ. 
  - Galera Replication Plugin: This plugin enables write-set replication service functionality.
  - Group Communication Plugins: There several group communication systems available to Galera Cluster (e.g., gcomm and Spread)

![](https://i.imgur.com/gvHRHgB.png)


![](https://i.imgur.com/WDMwP3u.png)


## LAB 
### Mô hình bài lab

Hệ điều hành: Centos 7
Database Management server: MariaDB 10.4

![](https://i.imgur.com/5ZThi84.png)


### Chuẩn bị

Thực hiện các bước sau trên tất cả các node:
- Thêm repo Mariadb 10.4:
```
cat << EOF > /etc/yum.repos.d/MariaDB.repo
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
EOF
```

- Cài đặt Mariadb:
```
yum install mariadb mariadb-server
```

- Cài đặt galera và một số gói cần thiết:
```
yum install galera-4 rsync
```
- Tắt mariadb:
```
systemctl stop mariadb
```

### Cấu hình trên node 1.
- Khởi tạo file cấu hình cho Galera trong thư mục **/etc/my.cnf.d/**:
```
cat <<EOF > /etc/my.cnf.d/galera.cnf
[galera]
# Mandatory settings
wsrep_on=ON
wsrep_provider=/usr/lib64/galera/libgalera_smm.so

#add your node ips here
wsrep_cluster_address="gcomm://192.168.30.171,192.168.30.176,192.168.30.177"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
#Cluster name

wsrep_cluster_name="galera_cluster"
# Allow server to accept connections on all interfaces.

bind-address=0.0.0.0

# this server ip, change for each server
wsrep_node_address="192.168.30.171"
# this server name, change for each server
wsrep_node_name="node1"

wsrep_sst_method=rsync
EOF
```

- Cấu hình firewalld cho phép các port 3306/tcp(mysql), 4567/tcp and udp(cho lưu lượng replication giữa các node), 4568/tcp(IST) và 4444/tcp(SST):
```
firewall-cmd --add-service=mysql --permanent
firewall-cmd --add-port={3306/tcp,4567/tcp,4567/udp,4568/tcp,4444/tcp} --permanent
firewall-cmd --reload
```

- Khởi tạo cluster:
```
galera_new_cluster
```

## Nguồn tài liệu
- https://github.com/hungnt1/Openstack_Research/blob/master/High-availability/4.%20Database/1.%20Galera-Cluster.md
- https://coffeecode101.blogspot.com/2016/01/gioi-thieu-galera-cluster.html#:~:text=Galera%20cluster%20l%C3%A0%20m%E1%BB%99t%20gi%E1%BA%A3i,c%C3%A1ch%20th%E1%BB%A9c%20c%C5%A9ng%20%C4%91%C6%A1n%20gi%E1%BA%A3n.
- https://galeracluster.com/library/documentation/architecture.html