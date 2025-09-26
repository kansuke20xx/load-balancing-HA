# Nginx for Hight Availability

Bước 1: cài đặt nginx và keepalived trên 2 máy chủ.

```
$ sudo apt update
$ sudo apt install nginx keepalived -y

```

Bước 2: cấu hình Ngix

Thiết lập Ngix trên cả 2 máy chủ để phục vụ cùng 1 nội dung, để đơn giản hay tạo 1 tệp html đặt trong thư mục gốc của mặc định của tài liệu:

```
$ echo "Hello from $(hostname)" | sudo tee /var/www/html/index.html
$ sudo systemctl restart nginx

```

Xác minh rằng Nginx đang phục vụ trên cả 2 máy chủ:

```
$ curl http://<ip_server>

```

`Lưu ý`: nếu khi chạy `curl http://<ip_server>` mà client nào đó không phản hồi lại thì hãy kiểm tra

```
$ sudo ufw status

```
nếu thấy chưa có 80/tcp thì hãy chạy lệnh:

```
$ sudo ufw allow 80/tcp
$ sudo ufw reload
$ sudo ufw status

```
Vậy là sẽ fix được lỗi ufw không mở cổng 80.

Bước 3: cấu hình keepalived

I. trên máy chủ chính

chạy lệnh sau:

```
$ sudo nano /etc/keepalived/keepalived.conf

```

thêm cấu hình sau vào file:

```
vrrp_instance VI_1 {
    state MASTER
    interface eth0interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 12345
    }

    virtual_ipaddress {
        192.168.1.100/24
    }
}

```

- state: Đặt MASTERtrên máy chủ chính.
- interface: Thay thế eth0 bằng giao diện mạng của máy chủ của bạn (sử dụng 'ip a' để tìm).
- virtual_router_id: ID duy nhất (trong ví dụ này là 51).
- priority: Đặt cao hơn trên máy chủ chính (100).
- virtual_ipaddress: Chỉ định địa chỉ IP ảo được chia sẻ (192.168.1.100).

khởi động lại Keepalived để áp dụng các thay đổi:

```
$ sudo systemctl restart Keepalived

```

II. trên máy chủ phụ 

làm tương tự thay `MASTER` thành `BACKUP`

```
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 90
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 12345
    }

    virtual_ipaddress {
        192.168.1.100/24
    }
}

```

rồi restart lại 

```
$ sudo systemctl restart keepalived 

```

Bước 4: check xem có hoạt động đúng không

Trên bất kỳ thiết bị nào trong cùng mạng, hãy ping địa chỉ IP ảo:

```
$ ping 192.168.1.100

```

Nó sẽ phản hồi từ máy chủ chính.

Dừng keepalived trên máy chính

```
$ sudo systemctl stop Keepalived

```

Kiểm tra xem máy chủ phụ có tiếp quản IP ảo hay không bằng cách ping lại.

Khởi động lại Keepalived trên máy chủ chính để khôi phục vai trò của nó:

```
$ sudo systemctl start Keepalived

```

[src](https://medium.com/@obaff/nginx-for-high-availability-7d25cda24a38) 



