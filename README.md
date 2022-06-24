![](https://i.imgur.com/1zrXQ7O.png)
## Cài đặt docker trên Ubuntu 18.04

    sudo apt-get update
>
    sudo apt-get install ca-certificates curl gnupg lsb-release
>
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
>
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
>
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose

## Cài đặt Wordpress bằng image từ trang chủ

Tải image wordpress từ trang chủ [Docker](http://https://hub.docker.com):

    docker pull wordpress

Sau đó chạy image ở bước trên:

    docker run --name wordpress -p 8080:80 -d wordpress

Kết quả:

![](https://i.imgur.com/SRWE1IP.png)

## Cài đặt MySQL bằng image từ trang chủ

Tải image wordpress từ trang chủ [Docker](http://https://hub.docker.com):

    docker pull mysql

Sau đó chạy image ở bước trên với `MYSQL_ROOT_PASSWORD` là mật khẩu của `root`:

    docker run --name mysql-server -e MYSQL_ROOT_PASSWORD=123 -d mysql:latest

Thực thi `mysql` với container vừa tạo:

    docker exec -it mysql-server bash
>
    mysql -u root -p

Kết quả:

![](https://i.imgur.com/3qtwmY9.png)

## Cài đặt phpmyadmin bằng image từ trang chủ

Tải image phpmyadmin từ trang chủ [Docker](http://https://hub.docker.com):

    docker pull phpmyadmin

Sau đó chạy image ở bước trên với `mysql-server` là tên container `mysql`:

    docker run --name phpmyadmin -d --link mysql-server:db -p 8000:80 phpmyadmin

Kết quả:

![](https://i.imgur.com/v3ZLYj8.png)

## Tạo Docker-compose để tự hóa việc cài đặt

Tạo 1 file `docker-compose.yml` với nội dung:

    version: "3.8"

    services:
        db:
            image: mysql
            container_name: mysql-server
            ports:
            - "3306:3306"
            restart: always
            environment:
                MYSQL_USER: wordpress
                MYSQL_ROOT_PASSWORD: 123
                MYSQL_PASSWORD: wordpress
                MYSQL_DATABASE: wordpress
            volumes:
            - my-db:/var/lib/mysql
        
        phpmyadmin:
            image: phpmyadmin
            container_name: phpmyadmin
            ports:
            - "8000:80"
            environment:
                MYSQL_ROOT_PASSWORD: 123
                PMA_HOST: db 
        
        wordpress:
            depends_on:
            - db
            image: wordpress
            container_name: wordpress
            ports:
            - "8080:80"
            restart: always
            environment:
                WORDPRESS_DB_HOST: db:3306
                WORDPRESS_DB_USER: wordpress
                WORDPRESS_DB_PASSWORD: wordpress
                WORDPRESS_DB_NAME: wordpress
            volumes:
            - wp-db:/var/www/html

    volumes: 
        my-db:
        wp-db:

Chạy file `docker-compose.yml` vừa tạo:

    docker-compose up -d

Kết quả sẽ được tương tự như khi cài đặt từng dịch vụ.

## Cài đặt Portainer để quản lý Docker

Tạo 1 phân vùng để lưu trữ dữ liệu của Portainer:

    docker volume create portainer_data

Cài đặt Portainer Server container:

    docker run -d -p 8001:8000 -p 9000:9000 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce

Kết quả:

![](https://i.imgur.com/OvzZvzm.png)

Giao diện quản lý của `Portainer`:

![](https://i.imgur.com/XVpN40G.png)
