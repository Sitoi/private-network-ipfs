# 搭建 IPFS 私有网络

## 环境

两台 Linux 设备，分别配置他们的ip地址为 192.168.1.244 和 192.168.1.246

## 步骤

- 修改挂载目录
- 通过 docker 创建 IPFS 容器
- 确保配置 IPFS API 以允许跨源（CORS）请求
- 生成并分发一个共享 key
- 移除默认的 boostrap 节点
- 添加节点创建网络
- 重启服务
- 查看邻居

### 修改挂载目录

> 该步骤可跳过，默认挂载目录为 项目目录下的 `data` 和 `staging` 文件夹

```yaml
version: "3"

services:

  ipfs_host:
    container_name: ipfs_host
    image: docker.io/ipfs/go-ipfs:latest
    restart: always
    volumes:
      - ./staging:/export                 # (可不修改)修改 挂载目录 ./staging
      - ./data:/data/ipfs                 # (可不修改)修改 挂载目录 ./data
    ports:
      - 4001:4001
      - 0.0.0.0:8080:8080
      - 0.0.0.0:5001:5001
```

### 通过 docker 创建 IPFS 容器

> 192.168.1.244 和 192.168.1.246 节点 创建 IPFS 容器

运行 make 命令

```bash
make up
```

登陆网页：http://192.168.1.244:5001/webui
登陆网页：http://192.168.1.246:5001/webui

创建容器到正常访问页面所要等待时间大约 `3-5` 分钟


### 确保配置 IPFS API 以允许跨源（CORS）请求

> 192.168.1.244 和 192.168.1.246 节点 配置跨源（CORS）请求

运行以下命令：

```bash
docker exec ipfs_host ipfs config --json API.HTTPHeaders.Access-Control-Allow-Origin '["*"]' 
docker exec ipfs_host ipfs config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT", "GET", "POST"]'
```

### 生成并分发一个共享 key

> 192.168.1.244 和 192.168.1.246 节点 配置跨源（CORS）请求

> 注：每个节点的 key 文件`内容保持一致`

- 生成key

```bash
go get -u github.com/Kubuxu/go-ipfs-swarm-key-gen/ipfs-swarm-key-gen
./ipfs-swarm-key-gen > swarm.key
```

- 分发 key 到每个节点

目标路径： 挂载目录的 `data` 文件夹内

```bash
cp swarm.key data/
```

### 移除默认的 boostrap 节点

> 192.168.1.244 和 192.168.1.246 节点 配置跨源（CORS）请求

```bash
docker exec ipfs_host ipfs bootstrap rm --all
```

### 添加节点创建网络

- 获取 192.168.1.244 节点 信息

```bash
docker exec ipfs_host ipfs id
```

```json
{
	"ID": "QmNtnCQiqMezQiwPfRwf7KE8BoM8mBmUYYm3XsXNfh8DL7",
	"PublicKey": "CAASpgIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDGwu8A5Xw4okQmkpufHLIjIO5ZhQGnCGzqD/OgcGE5MNfPe6pSurRQTI9AgQWtODJOGWqj7GBe1PgZXHIVkIhVjPIQftpcJJ/D6STJIzH9gGApc7SA8iIh2i9TaTontnvLuDswRj2hce2vWXQhh3DZ8ttv9rrPYfMcuM0tWs7klMQgt3C67prxgTd0esXm5DevtgJeHazimJcjNMBsAc9niKBgV0KFwxZPGouBBFibzF9jTWOC0qk52c33a4LcvKohLWnJbJGJ8mQj1oI9Srmo2SV37UbAVECxRvCDa0HhJkUUzkzpyCFUncZ03Mz0D6VTTCEoFKMBcahHM7ZuCKezAgMBAAE=",
	"Addresses": [
		"/ip4/127.0.0.1/tcp/4001/ipfs/QmNtnCQiqMezQiwPfRwf7KE8BoM8mBmUYYm3XsXNfh8DL7",
		"/ip4/172.19.0.2/tcp/4001/ipfs/QmNtnCQiqMezQiwPfRwf7KE8BoM8mBmUYYm3XsXNfh8DL7",
		"/ip6/::1/tcp/4001/ipfs/QmNtnCQiqMezQiwPfRwf7KE8BoM8mBmUYYm3XsXNfh8DL7"
	],
	"AgentVersion": "go-ipfs/0.4.19/52776a7",
	"ProtocolVersion": "ipfs/0.1.0"
}
```

- 192.168.1.246 节点添加 192.168.1.244 

```bash
docker exec ipfs_host ipfs bootstrap add /ip4/192.168.1.244/tcp/4001/ipfs/QmNtnCQiqMezQiwPfRwf7KE8BoM8mBmUYYm3XsXNfh8DL7
```

### 重启服务

> 192.168.1.244 和 192.168.1.246 节点 服务重启

```bash
docker restart ipfs_host
```

### 查看邻居

> 192.168.1.244 查看邻居

```bash
docker exec ipfs_host ipfs swarm peers
```

```bash
/ip4/192.168.1.246/tcp/4001/ipfs/QmVUF31gtrV9H3V1ndDCEGfKLahBNi8E9PT4AHj6m2zpjj
```

> 192.168.1.246 查看邻居

```bash
docker exec ipfs_host ipfs swarm peers
```

```bash
/ip4/192.168.1.244/tcp/4001/ipfs/QmNtnCQiqMezQiwPfRwf7KE8BoM8mBmUYYm3XsXNfh8DL7
```

## 测试

- 在 192.168.1.244 节点 添加文件

```bash
docker exec ipfs_host ipfs add /data/ipfs/version
```

```text
 2 B / 2 B  100.00%added QmaHbbushv2gYBUyofdm853cy1HTcNuinqagCfMjMdGmNw version
```

- 在 192.168.1.246 节点 用哈希值读取该文件

```bash
docker exec ipfs_host ipfs cat QmaHbbushv2gYBUyofdm853cy1HTcNuinqagCfMjMdGmNw
```

```text
7
```

搭建的私有网络可以正常使用。


## 附录

> 查看运行日志

```bash
docker logs -f ipfs_host
```

> 停止容器

```bash
docker stop ipfs_host
```

> 删除容器

```bash
docker rm ipfs_host
```

> 重启容器

```bash
docker restart ipfs_host
```

> 运行 IPFS 命令 

```bash
docker exec ipfs_host <ipfs cmd>
```