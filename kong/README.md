# KONG

Docker部署Kong和Konga。

## 参考

- https://github.com/Kong/docker-kong/tree/master/compose
- https://dev.to/vousmeevoyez/setup-kong-konga-part-2-dan


## Github地址
https://github.com/Galaxyobe/docker-composes/tree/master/kong

## 部署步骤

1. 修改配置环境

    修改.env文件
2. 挂载磁盘配置
    - 如果需要挂载postgres_data到外部磁盘，请执行如下命令：
        ```
        mkdir -p /var/lib/postgresql/data
        docker volume create --driver local \
          --opt type=none \
          --opt device=/var/lib/postgresql/data \
          --opt o=bind postgres_data
        ```
3. 部署
    ```
    docker-compose up -d
    ```
4. 登录KONGA

    默认：http://localhost:1337


## KONG管理API配置
参考：
  - https://getkong.org/docs/latest/secure-admin-api/#kong-api-loopback
  - https://docs.konghq.com/hub/kong-inc/key-auth/

配置完后可以在KONGA页面中的KeyAuth填写LoopBack。

1. 新建管理API服务
   
   服务名称：admin
    ```
    curl --location --request POST 'http://localhost:8001/services/' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "name": "kong-admin",
        "host": "localhost",
        "port": 8001
    }'
    ```
2. 新建管理API路由
    ```
    curl --location --request POST 'http://localhost:8001/services/admin/routes' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "paths": ["/admin"]
    }'
    ```
3. API服务启用KeyAuth
    ```
    curl -X POST http://localhost:8001/services/admin/plugins \
        --data "name=key-auth" 
    ```
4. API路由启用KeyAuth
    ```
    curl -X POST http://localhost:8001/routes/{admin的路由ID}/plugins \
        --data "name=key-auth" 
    ```
5. 新建Consumer
    ```
    curl --location --request POST 'http://localhost:8001/consumers/' \
    --form 'username=konga' \
    --form 'custom_id=cebd360d-3de6-4f8f-81b2-31575fe9846a'
    ```
6. 新建Key
    ```
    curl --location --request POST 'http://localhost:8001/consumers/konga/key-auth'
    ```
7. 使用Key
    ```
    curl http://localhost:8000/{proxy path}?apikey=<some_key>
    curl http://localhost:8000/{proxy path} \
    -H 'apikey: <some_key>'
    ```
8. 查询Keys
    ```
    curl -X GET http://localhost:8001/key-auths
    ```    
9. 删除Key
   - consumer: The id or username property of the Consumer entity to associate the credentials to.
   - id: The id attribute of the key credential object.
   ```
   curl -X DELETE http://kong:8001/consumers/{consumer}/key-auth/{id}
   ```