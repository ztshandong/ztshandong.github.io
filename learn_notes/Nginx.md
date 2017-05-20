# nginx.conf配合host文件，最好也看一眼tomcat
```sh
http {
     server {
        listen       80;
        server_name  www.site1name.com;

        location / {
            proxy_pass http://app1name:8080;
        }
    }
    
     server {
        listen       80;
        server_name  api.site1name.com;

        location / {
            proxy_pass http://app2name:8088;
        }
    }
    
    
     server {
        listen       80;
        server_name  www.site2name.com;

        location / {
            proxy_pass http://app3name:8080;
        }
    }
    
}
```

# host
```sh
127.0.0.1 		app1name
127.0.0.1		app2name
127.0.0.1       app3name
```