# PrimeVue Showcase Docker 部署指南

## 快速开始

### 使用 Docker Compose（推荐）

```bash
# 在 apps/showcase 目录下
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

访问 http://localhost:3000

### 使用 Docker 命令

```bash
# 构建镜像（在项目根目录）
docker build -f apps/showcase/Dockerfile -t primevue-showcase:latest .

# 运行容器
docker run -d \
  --name primevue-showcase \
  -p 3000:3000 \
  -e NODE_ENV=production \
  primevue-showcase:latest

# 查看日志
docker logs -f primevue-showcase

# 停止容器
docker stop primevue-showcase
docker rm primevue-showcase
```

## 从 GitHub Container Registry 拉取

```bash
# 拉取最新版本
docker pull ghcr.io/hllshiro/primevue-showcase:latest

# 运行
docker run -d \
  --name primevue-showcase \
  -p 3000:3000 \
  ghcr.io/hllshiro/primevue-showcase:latest
```

## 配置镜像访问权限

如果镜像是私有的，需要先登录：

```bash
# 使用 GitHub Personal Access Token 登录
echo $GITHUB_TOKEN | docker login ghcr.io -u hllshiro --password-stdin

# 然后拉取镜像
docker pull ghcr.io/hllshiro/primevue-showcase:latest
```

创建 Personal Access Token：
1. GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. 勾选 `read:packages` 权限
3. 生成并保存 token

## 生产环境部署

### 使用 Nginx 反向代理

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Kubernetes 部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: primevue-showcase
spec:
  replicas: 2
  selector:
    matchLabels:
      app: primevue-showcase
  template:
    metadata:
      labels:
        app: primevue-showcase
    spec:
      # 如果镜像是私有的，需要配置 imagePullSecrets
      # imagePullSecrets:
      # - name: ghcr-secret
      containers:
      - name: showcase
        image: ghcr.io/hllshiro/primevue-showcase:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: primevue-showcase
spec:
  selector:
    app: primevue-showcase
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

创建 Kubernetes Secret（如果镜像是私有的）：
```bash
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=hllshiro \
  --docker-password=$GITHUB_TOKEN \
  --docker-email=your-email@example.com
```

## 环境变量

- `NODE_ENV`: 运行环境（默认: production）
- `NITRO_PORT`: 服务端口（默认: 3000）
- `NITRO_HOST`: 监听地址（默认: 0.0.0.0）

## 故障排查

### 查看容器日志
```bash
docker logs primevue-showcase
```

### 进入容器调试
```bash
docker exec -it primevue-showcase sh
```

### 检查健康状态
```bash
curl http://localhost:3000
```
