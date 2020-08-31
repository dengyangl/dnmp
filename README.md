# dnmp
docker搭建lnmp环境，php 7.2 + nginx latest + mysql 5.7 + redis 4

## 注意
  使用前，请先安装 docker 和 docker-compose
  
##  克隆此包
  
    git clone https://github.com/dengyangl/dnmp.git

### docker-compose.yml文件简单说明
 
    mysql:
        image: mysql:5.7
        container_name: mysql
        hostname: mysql
        ports:
          - "3306:3306"                         # 格式 - 项目配置文件使用的端口(如laravel的.env):容器内使用的端口
        volumes:
          - ./mysql:/var/lib/mysql
          - ./my.cnf:/etc/mysql/conf.d/my.cnf   # 可以将文件中的[mysqld]和[mysqldump]下面的max_allowed_packet值设置为320M，[mysqld]下面的sql_mode设置为"NO_AUTO_VALUE_ON_ZERO"，防止*.sql文件过大无法上传导入
        environment:
          - MYSQL_ROOT_PASSWORD=mypassword      # 修改为自己的mysql数据库登录密码
    
      redis:
        image: redis:4
        container_name: redis
        hostname: redis
        command: redis-server /usr/local/etc/redis/redis.conf --requirepass mypassword  修改为自己的redis登录密码
        volumes:
          - ./redis/redis.conf:/usr/local/etc/redis/redis.conf
        ports:
          - "6379:6379"                         # 格式 - 项目配置文件使用的端口(如laravel的.env):容器内使用的端口
    
      php:
        build: ./php-fpm                        # Dockerfile文件所在的文件夹目录
        container_name: php
        hostname: php
        ports:
          - "9000:9000"
        links:
          - mysql:mysql
          - redis:redis
        volumes:
          - /volumes/data/source:/volumes/data/source       # 格式：宿主机的目录:容器内的目录，可以修改为代码文件夹
          - ./php-fpm/php.ini:/usr/local/etc/php/php.ini:ro     # (1)可以将文件中，error_reporting的值设置为 E_ALL & ~E_NOTICE & ~E_WARNING & ~E_DEPRECATED & ~E_STRICT (2)可以将display_errors设置为on，方便调试
          - ./php-fpm/cron/laravel:/var/spool/cron/crontabs/laravel
          - ./php-fpm/supervisor/program.conf:/etc/supervisor/conf.d/program.conf
        command: supervisord -c /etc/supervisor/supervisord.conf
        working_dir: /volumes/data/source
        
        # 注意，nginx和php挂载出来的宿主机目录一定要一致(/volumes/data/source)，不然没法访问php文件
    
      nginx:
        image: nginx:latest
        container_name: nginx
        hostname: nginx
        ports:
          - "80:80"                             # 格式：浏览器访问的端口:nginx配置文件(xxx.conf)使用的监听端口
        volumes:
          - /volumes/data/source:/volumes/data/source       # 格式：宿主机的目录:容器内的目录，可以修改为代码文件夹
          - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
          - ./nginx/conf.d/:/etc/nginx/conf.d/:ro
        links:
          - php:PHP-FPM                         # php：容器的别名；PHP-FPM：nginx容器里面使用的php名字，nginx发送给php需要用到的，默认即可
   
   
 ## 执行命令
 
    # 进入文件夹
        cd dnmp     
    
    # 先构建php的镜像(需要些时间)
        docker-compose build
    
    # 修改 docker-compose.yml 文件的代码目录
        vi docker-compose.yml 或者 vim docker-compose.yml
        把 /volumes/data/source           # 修改为你本机的代码目录
    
    # nginx配置文件的修改
        cp nginx/demo.conf nginx/conf.d/defualt.conf
        vi defualt.conf 或者 vim defualt.conf   # 修改监听端口(yml文件中nginx配置的端口,填写后面那个)，虚拟域名，工作目录，日志文件名字(容器的工作目录默认是 /www)
    
    # cd 到dnmp文件夹,执行
        docker-compose up -d
    
    注意：
    (1)如果没有目录的话，请自己创建相关目录
    (2)第一次执行需要花点时间下载镜像
    (3)docker-compose up -d 的时候可能一些服务没有跑起来，
       比如mysql，可能你给的dnmp目录权限不够，要给充足权限
       sudo chmod -R 777 dnmp
    (4)dnmp/php-fpm/supervisor/program.conf文件，stdout_logfile的worker.log文件，路径要指向到docker-compose.yml文件里面所配置的工作目录下(如: /source/worker.log，/source为工作目录)
    (5)在php的volumes中加入(使得在docker容器里面执行 date查询的时间与linux的时间相同)： - /etc/localtime:/etc/localtime
    (6)在php.ini文件里面设置：date.timezone = "Asia/Shanghai"，使得php代码获取的时间与系统相同
    (7)swoole的websocket服务监听的端口号，要配置在php容器的ports下面，nginx的*.conf文件例子：
        server {
            listen 80;
        
            server_name 127.0.0.1;
        
            root        /source/swoole_test/server;
            index       index.php index.html index.htm;
            charset     utf-8;
        
            access_log /var/log/nginx/swoole_test_ws.access.log main;
            error_log  /var/log/nginx/swoole_test_ws.error.log debug;
        
            location / {
                proxy_http_version 1.1;
                proxy_set_header Connection "keep-alive";
                proxy_set_header X-Real-IP $remote_addr;
                if (!-f $request_filename) {
                     proxy_pass http://PHP-FPM:8812;    #重要
                }
            }
        
        }
    
## 命令参考

    # 修改了Dockerfile文件的话，请执行
        docker-compose build
    
    # 修改了docker-compose.yml文件，或者nginx里面的.conf文件，请执行
    
    # 关闭 dnmp 服务
        docker-compose down
    
    # 开启 dnmp 服务
        docker-compose up -d
        
    # 进入容器
        ./exec.sh php
        ./exec.sh mysql
        ./exec.sh nginx
        ./exec.sh redis
        
    如果在命令行登录数据库报错：Warning: World-writable config file ‘/etc/my.cnf’ is ignored
    将my.cnf的权限改为755：chmod 755 my.cnf
 
 ## 说明
 
   (1)使用的都是官方的镜像。
   
   (2)php通过Dockerfile文件进行创建，默认已安装 redis,iconv,gd,zip,grpc,sockets,swoole,yaconf,mosquitto(mqtt),rdkafka,pcntl 等扩展和composer包管理工具，若还需要其它扩展，可在该文件中添加，然后执行docker-compose build命令
 
      mosquitto(mqtt)启动命令：mosquitto -c /etc/mosquitto/mosquitto.conf -d
 
   (3)Dockerfile的composer中, composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/ 与 composer config -g repo.packagist composer https://packagist.phpcomposer.com 等效(将composer换成国内镜像，加上 -g [--global] 进行全局配置，就不会去查找composer.json文件，导致报错)
 
   (4)docker-compose.yml文件中包含了phpMyAdmin，nodejs镜像(还有些问题)
   
   (5)如果是linux系统使用，则可以在docker-compose.yml文件的php镜像里面的volumes中加入 /etc/localtime:/etc/localtime (容器的时间时区与linux的同步)
 
   注意：此为基础版本，后续会继续完善
   
 
 ## 持续更新的内容
 
   1.2020.8.21
   
     (1)Dockerfile
        (1.1)librdkafka
        (1.2)php-rdkafka
        (1.3)ImageMagick(linux扩展)(pdf转png)
        (1.4)Imagick(php扩展)(pdf转png)
        (1.5)xvfb(linux扩展-html转pdf)
        (1.6)wkhtmltopdf(linux扩展-html转pdf) 
        
     (2)php.ini
        (2.1)max_execution_time = 300
             request_terminate_timeout = 300
        (2.2)memory_limit = 2048M
        (2.3)error_reporting = E_ALL & ~E_NOTICE & ~E_WARNING & ~E_DEPRECATED & ~E_STRICT
        (2.4)tp5：display_errors = On
        (2.5)upload_max_filesize = 20M
        (2.6);extension="phpwkhtmltox.so"
             ;extension=imagick.so
             
     (3)my.cnf
         (3.1)max_allowed_packet = 320M
         (3.2)sql_mode="NO_AUTO_VALUE_ON_ZERO"
         (3.3)max_allowed_packet = 320M
         
     (4)nginx.conf
         (4.1)#fastcgi方式
             #fastcgi 连接超时时间，默认60秒
             #fastcgi_connect_timeout 300;
             #nginx 进程向 fastcgi 进程发送请求过程的超时时间，默认值60秒
             #fastcgi_send_timeout 300;
             #fastcgi_read_timeout 300;
             #fastcgi_buffer_size 128k;
             #fastcgi_buffers 8 128k;
             #fastcgi_busy_buffers_size 256k;
             #fastcgi_temp_file_write_size 256k;
         (4.2)#proxy方式
             #proxy_connect_timeout 300;
             #proxy_send_timeout 300;
             #proxy_read_timeout 300;
             #proxy_buffer_size 16k;
             #proxy_buffers 4 64k;
             #proxy_busy_buffers_size 128k;
             #proxy_temp_file_write_size 128k;
             
     (5)thinkphp5_demo.conf
         (5.1)client_max_body_size 20m;
         
     (6)docker-compose.yml
         (6.1)sonarqube
         (6.2)postgres1
         (6.3)sentry1
         (6.4)cron
         (6.5)worker
         
   2.2020.8.31
   
      (1)Dockerfile
         (1.1)更换源
         (1.2)更换swoole版本(4.3.3--->4.5.2)
         (1.3)删除mosquitto-$MOSQUITTO_VERSION.tgz压缩包，librdkafka-master.zip压缩包，php-rdkafka-$PHP_RDKAFKA_VERSION.tar.gz压缩包
              
      (2)php.ini
         (2.1)swoole.use_shortname = off;   php的hyperf框架需要配置
       

 ## 联系我
   
   如果有什么问题或者想交流的，欢迎联系！
   
   email: 783973660@qq.com
  