# KONG

Docker 部署 Kong 和 Konga。

## 参考

- https://github.com/Kong/docker-kong/tree/master/compose
- https://dev.to/vousmeevoyez/setup-kong-konga-part-2-dan

## Github 地址

https://github.com/Galaxyobe/docker-composes/tree/master/kong

## 部署步骤

1. 修改配置环境

   修改.env 文件

2. 挂载磁盘配置
   - 如果需要挂载 postgres_data 到外部磁盘，请执行如下命令：
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
4. 登录 KONGA

   默认：http://localhost:1337

## KONG 管理 API 配置

参考：

- https://getkong.org/docs/latest/secure-admin-api/#kong-api-loopback
- https://docs.konghq.com/hub/kong-inc/key-auth/

配置完后可以在 KONGA 页面中的 KeyAuth 填写 LoopBack。

1. 新建管理 API 服务

   服务名称：kong-admin-api

   ```
   curl -X POST -H Content-Type:application/json -d '{"name": "kong-admin-api","host": "localhost","port": 8001}' http://localhost:8001/services/
   ```

2. 新建管理 API 路由
   ```
   curl -X POST -H Content-Type:application/json -d '{"paths": ["/kong-admin-api"]}' http://localhost:8001/services/kong-admin-api/routes
   ```
3. API 服务启用 KeyAuth
   ```
   curl -X POST http://localhost:8001/services/kong-admin-api/plugins \
       --data "name=key-auth"
   ```
4. API 路由启用 KeyAuth
   ```
   curl -X POST http://localhost:8001/routes/{kong-admin-api的路由ID}/plugins \
       --data "name=key-auth"
   ```
5. 新建 Consumer
   ```
   curl --location --request POST 'http://localhost:8001/consumers/' \
   --form 'username=konga' \
   --form 'custom_id=cebd360d-3de6-4f8f-81b2-31575fe9846a'
   ```
6. 新建 Key
   ```
   curl --location --request POST 'http://localhost:8001/consumers/konga/key-auth'
   ```
7. 使用 Key
   ```
   curl http://localhost:8000/{proxy path}?apikey=<some_key>
   curl http://localhost:8000/{proxy path} \
   -H 'apikey: <some_key>'
   ```
8. 查询 Keys
   ```
   curl -X GET http://localhost:8001/key-auths
   ```
9. 删除 Key
   - consumer: The id or username property of the Consumer entity to associate the credentials to.
   - id: The id attribute of the key credential object.
   ```
   curl -X DELETE http://kong:8001/consumers/{consumer}/key-auth/{id}
   ```

## 使用 KONG 代理 KONGA

1. 新建管理 API 服务

   服务名称：konga
   ```
   curl -X POST -H Content-Type:application/json -d '{"name": "konga","host": "konga","port": 1337}' http://localhost:8001/services/
   ```

2. 新建管理 API 路由
   在 hosts 中增加
   ```
   127.0.0.1 admin.kong.io
   ```
   ```
   curl -X POST -H Content-Type:application/json -d '{"hosts": ["admin.kong.io"]}' http://localhost:8001/services/konga/routes
   ```
   就可以使用 http://admin.kong.io 访问服务 KONGA
