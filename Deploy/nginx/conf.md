user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    client_max_body_size 10M;

    server {
        listen    80;
        listen    [::]:80;
        server_name  도메인 이름;
        listen 443 ssl;

        include /etc/nginx/default.d/*.conf;
        ssl_certificate /etc/letsencrypt/live/도메인/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/도메인/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf;

        set $is_redirect_val 0;

        if ($scheme != 'https') {
            set $is_redirect_val 1;
        }

        if ($host = 'localhost') {
            set $is_redirect_val 0;
        }

        if ($is_redirect_val = 1) {
            return 301 https://$host$request_uri;
        }

        if ($http_user_agent = "") {
            return 403;
        }

        if ($http_user_agent ~* (MJ12bot|AhrefsBot|SemrushBot|ltx71) ) {
            return 403;
        }

        location / {
            root /var/www/html/build;
            index  index.html index.html;
            try_files $uri $uri/ /index.html;
        }

        include /etc/nginx/conf.d/service-url.inc;

        location /api {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_pass $service_url;
        }

        location /oauth2 {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_pass $service_url;
        }

        location /login {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_pass $service_url;
        }

        location /docs {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_pass $service_url;
        }
    }


  server {
    listen       443 ssl;
    listen       [::]:443 ssl;
    server_name  dev2.minibeit.com;

    ssl_certificate /etc/letsencrypt/live/도메인/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/도메인/privkey.pem;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
            root /var/www/html/build;
            index  index.html index.html;
            try_files $uri $uri/ /index.html;
    }

    include /etc/nginx/conf.d/service-url.inc;

    location /api {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_pass $service_url;
    }

    location /oauth2 {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_pass $service_url;
    }

    location /login {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_pass $service_url;
    }

    location /docs {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_pass $service_url;
    }
  }
}
