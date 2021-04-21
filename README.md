frp内网穿透带web 管理 的docker 镜像
===

### 启动服务端 frps

```
docker run -itd -p 8080:8080 \
                       -p 7500:7500 \
                       -p 80:80 \
                       -p 443:443 \
                       -p 2000-3000:2000-3000 \
                       -p 7500:7500 \
                       -p 7001:7001/udp \
                       -e "HTTP_USER=admin" \
                       -e "HTTP_PASS=admin" \
                       --restart=always \
                       --name frps \
                       mtjo/frp:frps-v0.36.2
```
* frps 容器内配置文件可以通过 `-v /opt/frp/frps_full.ini:/etc/frps_full.ini` 可以挂载到本地实现持久化 
* 端口映射请根据实际配置文件修改
* 容器运行后可以通过你的 ip:8080管理服务端配置 http://XXX.XXX.XXX.XXX:8080

![frps](https://github.com/mtjo/webfrp/raw/master/images/frps.png)

### 在本地启动frpc客户端

```
docker run -itd -p 8080:8080 \
			    -p 7400:7400 \
			    -e "HTTP_USER=admin" \
			    -e "HTTP_PASS=admin" \
			    --restart=always \ 
			    --name frpc \
			    mtjo/frp:frpc-v0.36.2
```

* frpc 容器内配置文件可以通过` -v /opt/frp/frpc.ini:/etc/frpc.ini `可以挂载到本地实现持久化
* 通过你的[localhost:8080](localhost:8080)管理客户端配置 [http://127.0.0.1:8080](http://127.0.0.1:8080)

![frpc](https://github.com/mtjo/webfrp/raw/master/images/frpc.png)


## 当然也许没有合适你的frp版本你也可以自己通过dockerfile生成自己的镜像

### 生成frps (修改FRP_VERSION为对应版本号)
```
docker build -f DockerfileFrps --build-arg FRP_VERSION="0.36.2" -t mtjo/frp:frps-v0.36.2 .
```

### 生成frpc(修改FRP_VERSION为对应版本号)
```
docker build -f DockerfileFrpc --build-arg FRP_VERSION="0.36.2" -t mtjo/frp:frpc-v0.36.2 .
```