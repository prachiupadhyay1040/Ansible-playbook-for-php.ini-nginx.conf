# Ansible-playbook-for-php.ini-nginx.conf

Ansible playbook for php-nginx

Two file
1.cat nginx.conf 

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

2.cat default.conf 
server {
    listen 80;
    server_name localhost;
    root /var/www/html/code;

    location / {
        try_files $uri $uri/ /index.php?$uri&$args;
        index index.php index.html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass    localhost:9000;
        fastcgi_index   index.php;
        fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include         fastcgi_params;
        fastcgi_param   PATH_INFO $fastcgi_path_info;
    }

    location ~ /\.ht {
        deny all;
    }
}








===================================

cat playbook-nginix-php.yml 
---
  - hosts: 127.0.0.1
    tasks:

    - name: Create a directory if it does not exist
      file:
        path: /home/keenable/php-I
        state: directory
        owner: keenable
        group: keenable
        mode: '0755'
    - name: Create a directory if it does not exist
      file:      
        path: /home/keenable/php-nginx-default
        state: directory
        owner: keenable
        group: keenable
        mode: '0755'
       
    - name: Create a directory if it does not exist
      file:      
        path: /home/keenable/php-data
        state: directory
        owner: keenable
        group: keenable
        mode: '0755' 

    - name: Copy a new "php.ini" file into place, backing up the original if it differs from the copied version
      ansible.builtin.copy:
        src: /home/keenable/demo/php.ini
        dest: /home/keenable/php-I/php.ini
       
       
    - name: Copy a new "cis_code" file into place, backing up the original if it differs from the copied version
      ansible.builtin.copy:
        src: /home/keenable/demo/podman_volumes/cis
        dest: /home/keenable/php-data/cis  
         
    - name: Copy a new "default.conf" file into place, backing up the original if it differs from the copied version
      ansible.builtin.copy:
        src: /home/keenable/demo/default.conf
        dest: /home/keenable/php-nginx-default/default.conf

    - name: Connect random port from localhost to port 80 on pod2
      containers.podman.podman_pod:
        name: pod2
        state: started
        publish:
        - "80:80"
        - "9000:9000"
    - name: create a container
      containers.podman.podman_container:
        pod: pod2
        name: php1
        image: docker.io/library/php:8.0.5-fpm-alpine
        state: started
        detach: true
        volumes:
        - "/home/keenable/php-I/php.ini:/usr/local/etc/php/php.ini"
        - "/home/keenable/php-data/cis/cis/code:/var/www/html:rw"


    - name: create a container
      containers.podman.podman_container:
        pod: pod2
        name: nginx2
        image: docker.io/library/nginx:latest
        state: started
        detach: true
        volumes:
        - "/home/keenable/php-nginx-default/default.conf:/etc/nginx/conf.d/default.conf"
        - "/home/keenable/php-data/cis/cis/code:/var/www/html:rw"
        
        
        ==============================================================
        cat default.conf 
server {
    listen 80;
    server_name localhost;
    root /var/www/html/code;

    location / {
        try_files $uri $uri/ /index.php?$uri&$args;
        index index.php index.html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass    localhost:9000;
        fastcgi_index   index.php;
        fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include         fastcgi_params;
        fastcgi_param   PATH_INFO $fastcgi_path_info;
    }

    location ~ /\.ht {
        deny all;
    }
}

========================================================



        


