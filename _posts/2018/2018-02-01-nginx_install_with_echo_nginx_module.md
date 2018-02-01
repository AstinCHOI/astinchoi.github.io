---
layout: post
title: Nginx installation with echo-nginx-module
tags: [nginx, http, echo-nginx-module, echo_read_request_body]
---


### 1. Why post?
1.1. Sometimes I need to check **the HTTP logs** such as post body and some headers from **the cloud services.** So, I just choose the nginx for this.  

1.2. In order to install nginx modules, you have to install manually, not using package managements such as apt-get and yum.  


### 2. Installation
2.1. Install the echo-nginx-module module and its dependencies (Ubuntu)  


```bash
 $ sudo apt-get install build-essential zlib1g-dev libpcre3 libpcre3-dev unzip
```

2.2. Go to [Nginx Download Site](http://nginx.org/en/download.html) then download stable version.

2.3. Download [echo-nginx-module](https://github.com/openresty/echo-nginx-module/tags) from the [github](https://github.com/openresty/echo-nginx-module)

```bash
 $ wget http://nginx.org/download/nginx-1.12.2.tar.gz
 $ tar -xvzf nginx-1.12.2.tar.gz
 
 $ wget https://github.com/openresty/echo-nginx-module/archive/v0.61.zip
 $ unzip v0.61.zip
 
 $ cd nginx-1.12.2
 # you would install you nginx under /usr/local/nginx/.
 $ ./configure --prefix=/usr/local/nginx --add-module=../echo-nginx-module-0.61 --with-http_perl_module
 $ make -j2
 $ make install
```


### 3. Setting nginx.conf to see the http post log
```bash
$ cd /usr/local/nginx
$ sudo vim conf/nginx.conf
```

* See the request_body in "log_format" and if statement in "location /".

```nginx
http {
    # ...

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" ' 
                      '"$request_body"';

    # ...

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        access_log  logs/host.access.log  main;

        location / {
            default_type text/html;
            root   html;
            index  index.html index.htm;

            if ($request_method = POST) {
                error_page  405     =200 $uri;
                echo_read_request_body;
                echo $request_body;
            }
        }
    }
}
```


### 4. Run Nginx
```bash
 $ cd /usr/local/nginx
 
 # Test Configuration
 $ sudo sbin/nginx -t
 
 # Start nginx
 $ sudo sbin/nginx
 
 # $ sudo sbin/nginx -s reload
 # $ sudo sbin/nginx -s stop

 # Check the Process 
 $ ps aux | grep nginx
```

4.1. Then, test like this.

```bash
 $ curl -X POST http://localhost --data 'hello world'
```

And

[http://localhost](http://localhost) in your browser.

4.2. Finally Check the logs.
```bash
 $ cat /usr/local/nginx/logs/access.log
 $ cat /usr/local/nginx/logs/host.access.log
```


### 5. Further works
* Nginx preference directives
* Use more functions for 'echo-nginx-module'
* Other usuful nginx modules installation 
* Vim shortcut


### 6. Reference
6.1. https://github.com/openresty/echo-nginx-module#installation  
6.2. https://github.com/openresty/echo-nginx-module#echo_read_request_body  