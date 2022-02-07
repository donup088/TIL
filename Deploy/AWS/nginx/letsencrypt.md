## nginx에 let's encrypt 적용하기

### aws linux2 nginx에 let's encrypt 적용
- cerbot 설치
    - cerbot 설치에 필요한 EPEL7 저장소를 설치한 후 cerbot을 설치한다.
    ```
    sudo wget -r --no-parent -A 'epel-release-*.rpm' http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/
    sudo rpm -Uvh dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-*.rpm
    sudo yum-config-manager --enable epel*
    sudo yum repolist
    sudo yum install -y certbot
    sudo yum install certbot-nginx
    sudo service nginx stop
    sudo certbot --nginx
    ```
    - 도메인 입력과정에서 도메인을 입력해준다. ex)example.com

- nginx 설정 수정 
    - /etc/nginx/nginx.conf 경로에 가서 수정한다.
    ```
     server {
        listen       80;
        listen       [::]:80;
        server_name  도메인이름;
        listen 443 ssl;

        include /etc/nginx/default.d/*.conf;

        ssl_certificate /etc/letsencrypt/live/도메인 이름/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/도메인 이름/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf;

        if ($scheme != "https") {
            return 301 https://$host$request_uri;
        }

        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_pass jenkins ip 주소;
        }

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
    ```
    ```
    server {
        listen       443 ssl;
        listen       [::]:443 ssl;
        server_name  도메인이름;
        root         /usr/share/nginx/html;


        ssl_certificate /etc/letsencrypt/live/도메인이름/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/도메인이름/privkey.pem;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

         location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_pass jenkins ip 주소;
        }


        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
    ```
- letsencrypt는 3개월마다 유효기간이 만료된다. 알람메일이 오면 갱신하도록 하자.
    ```
    sudo certbot renew
    ```