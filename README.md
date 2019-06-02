# dnmp
docker搭建lnmp环境，php 7.2 + nginx latest + mysql 5.7 + redis 4

## 注意
  使用前，请先安装 docker 和 docker-compose
  
##  克隆此包
  
     git clone https://github.com/dengyangl/dnmp.git

### 修改docker-compose.yml
 
    mysql:
        image: mysql:5.7
        container_name: mysql
        hostname: mysql
        ports:
          - "3306:3306"
        volumes:
          - ./mysql:/var/lib/mysql
          - ./my.cnf:/etc/mysql/conf.d/my.cnf
        environment:
          - MYSQL_ROOT_PASSWORD=mypassword      修改为自己的mysql数据库登录密码
    
      redis:
        image: redis:4
        container_name: redis
        hostname: redis
        command: redis-server /usr/local/etc/redis/redis.conf --requirepass mypassword  修改为自己的redis登录密码
        volumes:
          - ./redis/redis.conf:/usr/local/etc/redis/redis.conf
        ports:
          - "6379:6379"
    
      php:
        build: ./php-fpm        # Dockerfile文件所在的文件夹目录
        container_name: php
        hostname: php
        ports:
          - "9000:9000"
        links:
          - mysql:mysql
          - redis:redis
        volumes:
          - /volumes/data/source:/volumes/data/source       # 格式：宿主机的目录:容器内的目录，可以修改为代码文件夹
          - ./php-fpm/php.ini:/usr/local/etc/php/php.ini:ro
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
          - "80:80"
        volumes:
          - /volumes/data/source:/volumes/data/source       # 格式：宿主机的目录:容器内的目录，可以修改为代码文件夹
          - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
          - ./nginx/conf.d/:/etc/nginx/conf.d/:ro
        links:
          - php:PHP-FPM             # php：容器的别名；PHP-FPM：nginx容器里面使用的php名字，nginx发送给php需要用到的，默认即可
   
   
 ## 修改完之后执行命令
  
    cd dnmp
    
    # 修改代码目录
    vi docker-compose.yml
    把 /volumes/data/source 修改为你本机的代码目录
    
    docker-compose build    构建php的镜像

    cp nginx/demo.conf nginx/conf.d/defualt.conf

    # vi defualt.conf   修改自己的监听端口，虚拟域名，工作目录，日志文件名字(容器的工作目录默认是 /www)
    
    docker-compose up -d
    
    如果没有目录的话，请自己创建相关目录
    
 第一次执行需要花点时间下载镜像
  
    # docker-compose up -d 的时候可能一些服务没有跑起来，
    # 比如mysql，可能你给的dnmp目录权限不够，要给充足权限
    
    sudo chmod -R 777 dnmp
    
## 命令参考

    # 修改了Dockerfile文件的话，请执行
        docker-compose build
    
    # 修改了docker-compose.yml文件，或者nginx里面的.conf文件，请执行
    
    # 关闭 dnmp 服务
        docker-compose down
    
    # 开启 dnmp 服务
        docker-compose up -d
 
 ## 说明
 
   使用的都是官方的镜像。
   
   php通过Dockerfile文件进行创建，默认已安装 redis,iconv,gd,grpc,sockets,swoole等扩展和composer包管理工具，若还需要其它扩展，可在该文件中添加，然后执行docker-compose build命令
 
   此为基础版本，后续会继续完善
 
 ## 联系我
   
   如果有什么问题或者想交流的，欢迎联系！
   
   email: 783973660@qq.com
   