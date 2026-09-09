# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 常用开发命令

### 构建 Docker 镜像
```bash
# 构建 frps（服务端）镜像
docker build -f DockerfileFrps --build-arg FRP_VERSION="<版本号>" -t <您的镜像标签> .

# 构建 frpc（客户端）镜像
docker build -f DockerfileFrpc --build-arg FRP_VERSION="<版本号>" -t <您的镜像标签> .
```
示例（使用仓库默认版本）：
```bash
docker build -f DockerfileFrps --build-arg FRP_VERSION="0.36.2" -t mtjo/frp:frps-v0.36.2 .
docker build -f DockerfileFrpc --build-arg FRP_VERSION="0.36.2" -t mtjo/frp:frpc-v0.36.2 .
```

### 运行容器
#### frps 服务端
```bash
docker run -itd \
  -p 8080:8080 -p 7500:7500 -p 80:80 -p 443:443 \
  -p 2000-3000:2000-3000 -p 7001:7001/udp \
  -e "HTTP_USER=admin" -e "HTTP_PASS=admin" \
  --restart=always --name frps \
  mtjo/frp:frps-v0.36.2
```
* 配置文件挂载（可选）：`-v /opt/frp/frps_full.ini:/etc/frps_full.ini`
* 管理界面：`http://<主机IP>:8080`

#### frpc 客户端
```bash
docker run -itd \
  -p 8080:8080 -p 7400:7400 \
  -e "HTTP_USER=admin" -e "HTTP_PASS=admin" \
  --restart=always --name frpc \
  mtjo/frp:frpc-v0.36.2
```
* 配置文件挂载（可选）：`-v /opt/frp/frpc.ini:/etc/frpc.ini`
* 管理界面：`http://localhost:8080`

### 注意事项
* 项目**不包含源码、测试、Lint 配置或构建脚本**，仅提供两个 Dockerfile 用于打包 FRP 二进制文件及 Web UI（webproc）。
* 所有版本依赖通过 `FRP_VERSION` 构建参数指定；若需其他版本，修改该参数即可。
* 构建基于 Alpine 镜像，入口点为 `webproc -c /etc/frpc.ini frpc -c /etc/frpc.ini`（客户端）或 `webproc -c /etc/frps_full.ini frps -c /etc/frps_full.ini`（服务端）。
* **CI/CD**：`.github/workflows/docker-image.yml` 在推送 `v*` tag、每日定时或手动触发时自动构建镜像；如需推送到 Docker Hub，请在仓库 `Settings → Secrets` 中配置 `DOCKER_USERNAME` 与 `DOCKER_TOKEN`。

## 项目结构与架构
```
/Users/mtjo/work/frp/
├── README.md              # 使用说明（含构建与运行命令）
├── DockerfileFrps         # frps 服务端镜像构建文件
├── DockerfileFrpc         # frpc 客户端镜像构建文件
├── .github/workflows/docker-image.yml  # GitHub Actions：自动构建镜像
├── .gitignore             # 忽略 .DS_Store
└── images/                # frps/frpc 管理界面截图（仅示意）
```

### 关键设计点
* **极简打包**：Dockerfile 仅完成三件事：
  1. 更改 APK 源为国内镜像并安装 tzdata、curl、bash；
  2. 下载并解压官方 FRP 预编译二进制包（Linux amd64）；
  3. 安装 webproc（用于提供 Web 管理界面）并设置为容器入口。
* **运行时配置**：FRP 的 TOML 配置文件在容器内默认路径为 `/etc/frpc.ini`（客户端）或 `/etc/frps_full.ini`（服务端），可通过 `-v` 挂载实现持久化与定制。
* **端口暴露**：
  * frps：`7000`（KCP 绑定），`7500`（控制端口），`8080`（Web UI），以及 HTTP/HTTPS/自定义端口映射；
  * frpc：`7400`（Web UI），`8080`（若需映射）。
* **无业务代码**：此仓库不包含任何可修改的源码；若需定制 FRP 功能，请在上游 fatedier/frp 仓库进行修改后，通过更换 `FRP_VERSION` 重新构建镜像。

> 以上信息均基于仓库现有文件（README、Dockerfile*、.gitignore）总结，供后续 Claude Code 会话直接参考。如需了解最新构建细节，请查看对应 Dockerfile 及 README 说明。