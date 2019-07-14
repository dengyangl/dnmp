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
          - ./my.cnf:/etc/mysql/conf.d/my.cnf
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
    (4)dnmp/php-fpm/supervisor/program.conf文件，stdout_logfile的worker.log文件，路径要指向到.yml文件所配置的工作目录下
    
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
 
   使用的都是官方的镜像。
   
   php通过Dockerfile文件进行创建，默认已安装 redis,iconv,gd,zip,grpc,sockets,swoole,yaconf等扩展和composer包管理工具，若还需要其它扩展，可在该文件中添加，然后执行docker-compose build命令
 
   此为基础版本，后续会继续完善
 
 ## 联系我
   
   如果有什么问题或者想交流的，欢迎联系！
   
   email: 783973660@qq.com
   