# APISIX Stream Proxy

Apisix proxy tcp/udp setup.

## Enabled stream
In conf/config.yaml uncomment stream_proxy section, apisix will listen on 6000 and 7000 port.
```yaml
stream_proxy:                 # TCP/UDP proxy
  tcp:                        # TCP proxy port list
    - 6000
  udp:                        # UDP proxy port list
    - 7000
```      

## Stream routes api
> get all stream routes
```
curl http://127.0.0.1:9080/apisix/admin/stream_routes -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X GET
```

> new stream route
```
curl http://127.0.0.1:9080/apisix/admin/stream_routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "server_port": 6000,
    "upstream": {
        "nodes": {
            "192.168.65.2:8010": 1
        },
        "type": "roundrobin"
    }
}'
```
visit apisix-ip:6000 will proxy to upstream nodes.

> remove stream route
```
curl http://127.0.0.1:9080/apisix/admin/stream_routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X DELETE
```
