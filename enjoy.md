# cấu hình load balancing (nginx)

Trước khi đi vào cấu hình thì tôi muốn nói rằng nó chỉ là một cấu hình cơ bản nhất mà trong quá trình học tập tôi làm, tôi cũng tham khảo các
nguồn trên mạng && các mô hình AI nữa. Chính vì vậy nếu có gì cần chỉnh sửa và góp ý để nó hoàn thiện hơn thì mọi người cứ đóng góp nhiệt tình
nhé. Còn bây giờ thì bắt tay vào làm thôi.

## Mô tả sơ qua những gì mà tôi sẽ cấu hình nhé:

- khi người dùng gửi request đến server thì thay vì gửi trực tiếp đến IP của server thì sẽ gửi đến 1 IP ảo, IP ảo này cũng là mấu chốt để liên 
kết các node lại với nhau để khi 1 node này chết thì sẽ chuyển tiếp request đến các node còn lại thông qua IP ảo đó chứ không bị lỗi hiển thị
với người dùng.

I. Cấu hình trên máy server (chứa dữ liệu để trả về client)

Cài Nginx:

```
$ sudo apt update
$ sudo apt install nginx -y

```
Cấu hình block mặc định `/etc/nginx/sites-enable/test_web_lab`

```
server {
    listen 80;
    server_name test_web_lab www.test_web_lab;
    root /var/www/test_web_lab;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}

```

Cấu hình block trong `/var/www/test_web_lab`

Tạo file html hoặc php đều được nếu có file html nó sẽ ưu tiên load trước, ở đây tôi sẽ tạo file php:
- /var/www/test_web_lab/index.php

```
<?php
echo "<h1>Welcome to test_web_lab</h1>";

```
Lưu và khởi động lại:

```
$ sudo systemctl restart nginx

```

Test:

```
$ curl http://<ip_server>

```

II. Cấu hình đến 2 máy node

Cài nginx + keepalived:

```
$ sudo apt update
$ sudo apt install nginx keepalived -y

```

Cấu hình trong `/etc/nginx/nginx.conf`

- 2 máy node cấu hình y hệt nhau nhé

```

user www-data; # Chạy Nginx dưới user www-data (tài khoản có quyền hạn hạn chế, bảo mật tốt hơn).
worker_processes auto; # Tự động chọn số process worker theo số CPU core → tận dụng tối đa tài nguyên máy.
pid /run/nginx.pid; # Lưu PID (process ID) của tiến trình chính vào file /run/nginx.pid, dùng để quản lý (stop, reload).

events {
    worker_connections 1024; # Mỗi worker process có thể mở tối đa 1024 kết nối đồng thời. Điều này ảnh hưởng trực tiếp đến số lượng client có thể kết nối song song.
}

http {

    #include /etc/nginx/mime.types; → định nghĩa các loại MIME (vd .html → text/html, .png → image/png…), giúp Nginx trả về đúng content-type.
    #default_type application/octet-stream; → loại mặc định nếu file không xác định được MIME (trả về như file nhị phân).
    #client_max_body_size 2M; → giới hạn size request body (vd file upload) tối đa 2 MB.
    
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    client_max_body_size 2M;
    

    #Định nghĩa một nhóm server (tên là backend).
    #Ở đây chỉ có 1 server duy nhất: 192.168.1.100:80.
    #Bình thường khi load balancing, bạn sẽ có nhiều server ...; trong upstream.
    #Ở đây dùng VIP .100 → tức là LB sẽ proxy request đến VIP quản lý bởi Keepalived.
    upstream backend {
        server 192.168.1.100:80;
    }
    


    #listen 80; → Lắng nghe trên cổng HTTP 80.
    #server_name _; → _ nghĩa là bắt mọi domain không khớp (mặc định).
    server {
        listen 80;
        server_name _;
        

        #location / { ... } → xử lý tất cả request vào /.
        #proxy_pass http://backend; → chuyển tiếp request đến upstream backend (ở trên, tức là 192.168.1.100:80).
        #sub_filter '</body>' '<p>Served by lb_01</p></body>';
        #Tìm trong response trả về đoạn </body>
        #Chèn thêm dòng <p>Served by lb_01</p> trước khi đóng body
        #Giúp bạn biết response đi qua LB nào.
        #sub_filter_once off; → áp dụng thay thế cho mọi lần xuất hiện của </body>, không chỉ lần đầu.
        location / {
            sub_filter '</body>' '<p>Served by lb_01</p></body>';
            sub_filter_once off;
            proxy_pass http://backend;
        }
    }
}

```

Tiếp tục cấu hình trong `/etc/keepalived/keepalived.conf`

- node 1:

```
vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight 50
}

vrrp_instance VI_1 {
    state MASTER
    interface ens        # chỉnh theo card mạng thật
    virtual_router_id 51
    priority 110
    advert_int 1
    virtual_ipaddress {
        192.168.1.xxx     # VIP để client truy cập 
    }
    track_script {
        check_nginx
    }
}

```
> xxx kia là số gì cũng được miễn là nằm trong giải mạng là được.

- node 2:

```
vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight 50
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens        # chỉnh theo card mạng thật
    virtual_router_id 51
    priority 100
    advert_int 1
    virtual_ipaddress {
        192.168.1.xxx     # VIP để client truy cập
    }
    track_script {
        check_nginx
    }
}

```
>> xxx này phải giống xxx ở node 1 nhé.

III. Thêm script để check nginx

```
#!/bin/sh
if [ -z "`pidof nginx`" ]; then
    systemctl stop keepalived
    exit 1
fi

```
Cấp quyền thực thi file:

```
$ sudo chmod +x /etc/keepalived/check_nginx.sh

``` 

Mục đích của script này là:

- Nếu nginx ở máy node 1 chết thì nó sẽ bắt lấy tín hiệu và dừng luôn keepalived ở node 1 đó và sẽ nhả cái IP mà client đang dùng 
để gửi request đến máy server.

- Khi đó thì node 2 kia (dự phòng) sẽ bắt lấy cái IP này và tiếp tục công việc của node 1 bị chết kia.

IV. Khởi động dịch vụ

Trên cả 2 node:

```
$ sudo systemctl enable nginx keepalived
$ sudo systemctl start nginx keepalived

```
> Lưu ý: nginx phải chạy trước khi keepalived chạy

V. Kiểm tra xem thành công hay không

```
curl http://<ip_server>

```

Nếu trả về nội dung ở file php hoặc html là thành công rồi.


