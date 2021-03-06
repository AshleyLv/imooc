## WebPageTest本地部署说明

1. 拉取镜像
    ```
    docker pull webpagetest/server
    
    docker pull webpagetest/agent
    ```

2. 运行实例
    ```
    docker run -d -p 4000:80 --rm webpagetest/server
    
    docker run -d -p 4001:80 --network="host" -e "SERVER_URL=http://localhost:4000/work/" -e "LOCATION=Test" webpagetest/agent
    ```
### mac 用户自定义镜像
1. 创建server目录
    ```
   mkdir wpt-mac-server
   cd wpt-mac-server
    ```
2. 创建Dockerfile，添加内容
    ```
   vim Dockerfile
   
   FROM webpagetest/server
   ADD locations.ini /var/www/html/settings/
    ```
3. 创建locations.ini配置文件，添加内容
    ```
   vim locations.ini
   
   [locations]
   1=Test_loc
   [Test_loc]
   1=Test
   label=Test Location
   group=Desktop
   [Test]
   browser=Chrome,Firefox
   label="Test Location"
   connectivity=LAN
    ```
4. 创建自定义server镜像
    ```
   docker build -t local-wptserver .
    ```
5. 创建agent目录
    ```
   mkdir wpt-mac-agent
   cd wpt-mac-agent
    ```
6. 创建Dockerfile，添加内容
    ```
   vim Dockerfile
   
   FROM webpagetest/agent
   ADD script.sh /
   ENTRYPOINT /script.sh
    ```
7. 创建script.sh， 添加内容
    ```
   vim script.sh
   
   #!/bin/bash
   set -e
   if [ -z "$SERVER_URL" ]; then
     echo >&2 'SERVER_URL not set'
     exit 1
   fi
   if [ -z "$LOCATION" ]; then
     echo >&2 'LOCATION not set'
     exit 1
   fi
   EXTRA_ARGS=""
   if [ -n "$NAME" ]; then
     EXTRA_ARGS="$EXTRA_ARGS --name $NAME"
   fi
   python /wptagent/wptagent.py --server $SERVER_URL --location $LOCATION $EXTRA_ARGS --xvfb --dockerized -vvvvv --shaper none
    ```
8. 修改script.sh权限
    ```
   chmod u+x script.sh
    ```   
9. 用新镜像运行实例 (注意先停掉之前运行的containers)
    ```
   docker run -d -p 4000:80 --rm wpt-mac-server
       
   docker run -d -p 4001:80 --network="host" -e "SERVER_URL=http://localhost:4000/work/" -e "LOCATION=Test" wpt-mac-agent
    ```   
