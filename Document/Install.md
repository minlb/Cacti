# Cài đăt, cấu hình, monitor và cảnh báo trên Cacti

* Địa chỉ IP server cài Cacti-server: 172.16.20.10
* Địa chỉ IP giám sát Linux server: 172.16.20.20 

# Cài đăt

## I. Giới thiệu

 * Cacti là một hệ thống monitor được thiết kế để giám sát các máy chủ và thiết bị mạng trong hệ thống.

 * Cacti được thiết kế để hoạt động dựa trên nền tảng là RRD Tool - một nền tảng cho phép theo dõi trên hệ thống tuy nhiên lại không có giao diện đồ họa. Cacti sẽ lấy các kết quả từ RRD Tool và chuyển thành giao diện dễ hiểu đối với người dùng.

 * Cacti được viết dựa trên ngôn ngữ chính là PHP và Perl, CSDL chính là MySQL, hoạt động trên giao diện Web, vì vậy để dùng Cacti cần phải cài đặt nó trên một Web server hỗ trợ PHP và MySQL.

 
## II. Cài đăt Cacti

 * Bước 1: Cài đặt web server (Apache, Mysql, PHP)
 			
		   	yum -y install httpd httpd-devel mysql mysql-server php-mysql php-pear php-common php-gd php-devel php php-mbstring php-cli php-mysql php-snmp
 * Bước 2: Cài đặt SNMP và RRD Tool để giám sát và vẽ đồ thị
 
			yum -y install net-snmp-utils net-snmp-libs php-pear-Net-SMTP rrdtool
 
 * Bước 3: Cấu hình Apache, Mysql chuẩn bị cho cài đặt cacti
 

 	- Cấu hình cho phép các dịch vụ tự khởi động khi server khởi động và start các service

				chkconfig httpd on 
				chkconfig mysqld on
				chkconfig snmpd on
	
				service httpd start
				service mysqld start
				service snmpd start
				
    - Cấu hình tạo mysql account và database cho quá trình cài đặt cacti
     
				[root@centos6 ~]# /usr/bin/mysql_secure_installation
				Enter current password for root (enter for none):
				Set root password? [Y/n] Y
				Remove anonymous users? [Y/n] Y
				Disallow root login remotely? [Y/n] n
				Remove test database and access to it? [Y/n] Y
				Reload privilege tables now? [Y/n] Y

	- Tạo account và database cacti sau đó cấp quyền cho cacti mysql account to toàn quyền trên database cacti bằng MySQL root account
	 
				[root@centos6 ~]# mysql -uroot -p
				Enter password:
				mysql> CREATE DATABASE cacti;
				
				mysql> CREATE USER 'cacti'@'localhost' IDENTIFIED BY 'password';
				Query OK, 0 rows affected (0.00 sec)
				
				mysql> GRANT ALL ON cacti.* TO 'cacti'@'localhost';
				Query OK, 0 rows affected (0.00 sec)
				
				mysql> FLUSH PRIVILEGES;
				Query OK, 0 rows affected (0.00 sec)
				
				mysql> exit

 * Bước 4: Cài đặt Cacti 
 
	- Vào trang http://www.cacti.net để download gói Cacti mới nhất trong mục Latest Files về giải nén trong thư mục /var/www/html
	   
 				cd /var/www/html/ 
				wget http://www.cacti.net/downloads/cacti-0.8.8h.tar.gz
				tar -xzvf cacti-0.8.8h.tar.gz
				mv cacti-0.8.8h cacti

	- Import database mẫu cho "cacti" database tạo trước đó 

                mysql -u root -p cacti < /var/www/html/cacti/cacti.sql  //(Nhập password root MySQL)
	- Chỉnh sửa username & password trong file cấu hình của cacti 

				vim /var/www/html/cacti/include/config.php
				/* make sure these values refect your actual database/host/user/password */
				$database_type = "mysql";
				$database_default = "cacti";
				$database_hostname = "localhost"; //có thể thay thế bằng địa chỉ 127.0.0.1 hoặc địa chỉ cài cacti-server
				$database_username = "cacti";
				$database_password = "password";
				$database_port = "3306";
				$database_ssl = false;

Hoàn thành xong cài đặt