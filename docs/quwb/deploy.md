# TradingAgents-CN 部署手册

**版本**: v1.0.1
**最后更新**: 2026-05-13
**维护者**: TradingAgents-CN 项目组

---

## 目录

- [1. 部署概述](#1-部署概述)
- [2. 热身准备](#2-热身准备)
  - [2.1 硬件要求](#21-硬件要求)
  - [2.2 软件要求](#22-软件要求)
  - [2.3 API密钥准备](#23-api密钥准备)
- [3. Docker部署（推荐）](#3-docker部署推荐)
  - [3.1 快速开始](#31-快速开始)
  - [3.2 详细部署步骤](#32-详细部署步骤)
  - [3.3 生产环境部署](#33-生产环境部署)
- [4. 本地代码部署](#4-本地代码部署)
  - [4.1 Python环境搭建](#41-python环境搭建)
  - [4.2 数据库安装](#42-数据库安装)
  - [4.3 后端部署](#43-后端部署)
  - [4.4 前端部署](#44-前端部署)
- [5. 配置说明](#5-配置说明)
- [6. 验证部署](#6-验证部署)
- [7. 数据同步](#7-数据同步)
- [8. 常见问题](#8-常见问题)
- [9. 性能优化](#9-性能优化)
- [10. 监控与维护](#10-监控与维护)

---

## 1. 部署概述

TradingAgents-CN 是一个前后端分离的多智能体股票分析系统，主要组件包括：

| 组件 | 技术栈 | 端口 | 说明 |
|------|--------|------|------|
| **后端服务** | FastAPI + Python 3.10 | 8000 | 提供RESTful API和分析服务 |
| **前端服务** | Vue 3 + Vite + Nginx | 3000 | Web用户界面 |
| **MongoDB** | 4.4 | 27017 | 主数据库，存储股票数据和分析结果 |
| **Redis** | 7-alpine | 6379 | 缓存和会话管理 |

### 部署方式对比

| 部署方式 | 难度 | 适用场景 | 推荐指数 |
|---------|------|---------|---------|
| **Docker部署** | ⭐⭐ 中等 | 生产环境、快速部署 | ⭐⭐⭐⭐⭐ 强烈推荐 |
| **本地代码部署** | ⭐⭐⭐ 较难 | 开发环境、定制需求 | ⭐⭐⭐ 开发者 |

---

## 2. 热身准备

### 2.1 硬件要求

#### Docker部署（推荐配置）

| 使用规模 | CPU | 内存 | 硬盘 | 说明 |
|---------|-----|------|------|------|
| **个人使用** | 4核 | 8GB | 50GB | 1-5并发分析 |
| **小团队** | 8核 | 16GB | 100GB | 5-20并发分析 |
| **生产环境** | 16核+ | 32GB+ | 200GB+ | 20+并发分析 |

#### 本地开发

- CPU: 4核以上
- 内存: 8GB以上
- 硬盘: 50GB以上可用空间
- 操作系统: Windows 10+, macOS 11+, Linux (Ubuntu 20.04+)

### 2.2 软件要求

#### Docker部署

- **Docker**: 20.10.0+
- **Docker Compose**: 2.0.0+

检查版本：
```bash
docker --version
docker-compose --version
```

#### 本地代码部署

- **Python**: 3.10.x （必须3.10，不支持3.11+）
- **Node.js**: 18.0.0+ （前端构建）
- **MongoDB**: 4.4+ （或使用Docker运行）
- **Redis**: 6.0+ （或使用Docker运行）

检查版本：
```bash
python --version  # 应该显示 Python 3.10.x
node --version    # 应该显示 v18.0.0 或更高
```

### 2.3 API密钥准备

在部署之前，请准备以下API密钥（至少需要其中一个LLM提供商）：

#### 必需的API密钥

| 服务 | 获取地址 | 用途 | 费用 |
|------|---------|------|------|
| **DeepSeek** | https://platform.deepseek.com/ | AI分析（推荐） | 按量付费，性价比高 |
| **阿里百炼** | https://dashscope.aliyun.com/ | AI分析（国产） | 按量付费 |
| **AIHubMix** | https://aihubmix.com/?aff=2rIi | 聚合LLM平台 | 按量付费，支持多模型 |

#### 可选的API密钥

| 服务 | 获取地址 | 用途 |
|------|---------|------|
| **Tushare** | https://tushare.pro/weborder/#/login?reg=tacn | 专业A股数据 |
| **FinnHub** | https://finnhub.io/ | 美股数据 |
| **OpenAI** | https://platform.openai.com/ | GPT模型 |
| **Google AI** | https://ai.google.dev/ | Gemini模型 |

#### 免费数据源

- **AKShare**: 完全免费，无需API密钥，支持A股数据
- **BaoStock**: 免费注册，有限制的A股数据

---

## 3. Docker部署（推荐）

Docker部署是最简单、最可靠的方式，适合绝大多数场景。

### 3.1 快速开始

#### 1. 克隆项目

```bash
# 使用Git克隆
git clone https://github.com/hsliuping/TradingAgents-CN.git
cd TradingAgents-CN

# 或下载ZIP包解压
```

#### 2. 配置环境变量

```bash
# 复制环境变量模板
cp .env.docker .env

# 编辑.env文件，填入你的API密钥
# Windows: notepad .env
# Linux/Mac: nano .env 或 vim .env
```

**最小配置示例**（.env文件）：

```env
# ==================== 必需配置 ====================

# MongoDB 配置（使用Docker内部网络）
MONGODB_HOST=mongodb
MONGODB_PORT=27017
MONGODB_USERNAME=admin
MONGODB_PASSWORD=tradingagents123
MONGODB_DATABASE=tradingagentscn
MONGODB_AUTH_SOURCE=admin

# Redis 配置（使用Docker内部网络）
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=tradingagents123
REDIS_DB=0

# JWT 安全配置（生产环境务必修改）
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_ALGORITHM=HS256
CSRF_SECRET=your-csrf-secret-key-change-in-production

# ==================== LLM配置（至少配置一个） ====================

# DeepSeek API（推荐）
DEEPSEEK_API_KEY=sk-your-deepseek-api-key
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_ENABLED=true

# 或使用阿里百炼
DASHSCOPE_API_KEY=sk-your-dashscope-api-key

# 或使用AIHubMix聚合平台
AIHUBMIX_API_KEY=sk-your-aihubmix-key
AIHUBMIX_BASE_URL=https://aihubmix.com/v1

# ==================== 数据源配置 ====================

# 默认中国股票数据源（推荐akshare，免费）
DEFAULT_CHINA_DATA_SOURCE=akshare

# Tushare（可选，专业数据）
TUSHARE_TOKEN=your_tushare_token
TUSHARE_ENABLED=false

# FinnHub（可选，美股数据）
FINNHUB_API_KEY=your_finnhub_key
```

#### 3. 启动服务

```bash
# 启动所有服务（后台运行）
docker-compose up -d

# 查看启动日志
docker-compose logs -f

# 等待所有服务健康检查通过（约1-2分钟）
# 看到 "All services are healthy!" 表示启动成功
```

#### 4. 访问应用

- **前端界面**: http://localhost:3000
- **后端API**: http://localhost:8000
- **API文档**: http://localhost:8000/docs

#### 5. 初始化系统

首次访问需要：

1. 注册管理员账号
2. 配置LLM提供商（设置 → LLM配置）
3. 初始化聚合渠道（如果使用AIHubMix等聚合平台）
4. 添加模型目录（选择要使用的模型）

### 3.2 详细部署步骤

#### 步骤1: 系统检查

```bash
# 检查Docker是否正常工作
docker run --rm hello-world

# 检查Docker Compose是否正常
docker-compose version

# 检查端口占用（确保3000, 8000, 27017, 6379端口未被占用）
# Windows: netstat -ano | findstr ":3000"
# Linux/Mac: lsof -i :3000
```

#### 步骤2: 拉取镜像（可选）

```bash
# 如果使用预构建镜像（推荐，节省构建时间）
docker pull ghcr.io/hsliuping/tradingagents-backend:v1.0.1
docker pull ghcr.io/hsliuping/tradingagents-frontend:v1.0.1

# 修改docker-compose.yml，将build改为image
# image: ghcr.io/hsliuping/tradingagents-backend:v1.0.1
```

#### 步骤3: 构建镜像（如果不使用预构建镜像）

```bash
# 构建后端镜像（首次构建需要10-20分钟）
docker-compose build backend

# 构建前端镜像（首次构建需要5-10分钟）
docker-compose build frontend

# 或同时构建所有镜像
docker-compose build
```

#### 步骤4: 数据持久化配置

```bash
# 创建数据目录（如果使用本地卷）
mkdir -p data/mongodb data/redis logs

# 修改docker-compose.yml，使用命名卷而不是匿名卷
volumes:
  mongodb_data:
    driver: local
    name: tradingagents_mongodb_data  # 使用命名卷
  redis_data:
    driver: local
    name: tradingagents_redis_data
```

#### 步骤5: 网络配置

```bash
# 检查Docker网络
docker network ls

# 创建自定义网络（可选）
docker network create tradingagents-network

# 修改docker-compose.yml使用自定义网络
networks:
  tradingagents-network:
    external: true
```

#### 步骤6: 启动顺序

```bash
# 1. 先启动数据库服务
docker-compose up -d mongodb redis

# 2. 等待数据库健康检查通过
docker-compose logs -f mongodb
# 看到 "waiting for connections on port 27017" 表示成功

# 3. 启动后端服务
docker-compose up -d backend

# 4. 等待后端健康检查通过
docker-compose logs -f backend
# 看到 "Application startup complete" 表示成功

# 5. 启动前端服务
docker-compose up -d frontend

# 6. 检查所有服务状态
docker-compose ps
```

#### 步骤7: 验证部署

```bash
# 检查服务健康状态
curl http://localhost:8000/api/health
curl http://localhost:3000/health

# 查看容器日志
docker-compose logs backend --tail 100
docker-compose logs frontend --tail 100

# 进入容器检查
docker exec -it tradingagents-backend bash
docker exec -it tradingagents-mongodb mongo
```

### 3.3 生产环境部署

#### 1. 使用Nginx反向代理

创建 `nginx-proxy.conf`：

```nginx
upstream backend {
    server localhost:8000;
}

upstream frontend {
    server localhost:3000;
}

server {
    listen 80;
    server_name your-domain.com;

    # 前端
    location / {
        proxy_pass http://frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # 后端API
    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # SSE支持
        proxy_buffering off;
        proxy_cache off;
        proxy_set_header Connection '';
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
    }

    # WebSocket支持
    location /ws/ {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### 2. 配置HTTPS（使用Let's Encrypt）

```bash
# 安装certbot
apt-get install certbot python3-certbot-nginx

# 获取证书
certbot --nginx -d your-domain.com

# 自动续期
certbot renew --dry-run
```

#### 3. 环境变量安全

```bash
# 使用Docker Secrets（Swarm模式）
echo "your-secret-key" | docker secret create jwt_secret -

# 或使用环境变量文件（不要提交到Git）
echo ".env" >> .gitignore
chmod 600 .env
```

#### 4. 资源限制

修改 `docker-compose.yml`：

```yaml
services:
  backend:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 8G
        reservations:
          cpus: '2'
          memory: 4G

  mongodb:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
```

#### 5. 日志管理

```yaml
services:
  backend:
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "5"
        compress: "true"
```

---

## 4. 本地代码部署

本地部署适合开发者，需要更多配置工作。

### 4.1 Python环境搭建

#### 1. 安装Python 3.10

**Windows**:
```
1. 访问 https://www.python.org/downloads/
2. 下载Python 3.10.x安装包
3. 安装时勾选"Add Python to PATH"
```

**Linux**:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3.10 python3.10-venv python3-pip

# CentOS/RHEL
sudo yum install python3.10 python3.10-pip
```

**macOS**:
```bash
# 使用Homebrew
brew install python@3.10
```

#### 2. 创建虚拟环境

```bash
# 进入项目目录
cd TradingAgents-CN

# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
# Windows:
venv\Scripts\activate

# Linux/Mac:
source venv/bin/activate

# 升级pip
pip install --upgrade pip
```

#### 3. 安装Python依赖

```bash
# 安装项目依赖
pip install -r requirements.txt

# 或使用pyproject.toml
pip install -e .
```

#### 4. 安装系统依赖

**Ubuntu/Debian**:
```bash
sudo apt-get update
sudo apt-get install -y \
    curl \
    pandoc \
    wkhtmltopdf \
    fonts-noto-cjk \
    mongodb \
    redis-server
```

**Windows**:
```bash
# 下载并安装
# Pandoc: https://pandoc.org/installing.html
# wkhtmltopdf: https://wkhtmltopdf.org/downloads.html

# 安装MongoDB
# 下载MSI安装包: https://www.mongodb.com/try/download/community

# 安装Redis（使用WSL或Windows版本）
# 下载: https://github.com/microsoftarchive/redis/releases
```

### 4.2 数据库安装

#### MongoDB安装与配置

**Linux**:
```bash
# 安装MongoDB
sudo apt-get install -y mongodb

# 启动服务
sudo systemctl start mongodb
sudo systemctl enable mongodb

# 创建应用数据库和用户
mongo
```

在MongoDB shell中执行：
```javascript
use tradingagentscn
db.createUser({
  user: "tradingagents",
  pwd: "your-password",
  roles: ["readWrite"]
})
```

**使用Docker运行数据库**（推荐）:

```bash
# MongoDB
docker run -d \
  --name tradingagents-mongodb \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=tradingagents123 \
  -v mongodb_data:/data/db \
  mongo:4.4

# Redis
docker run -d \
  --name tradingagents-redis \
  -p 6379:6379 \
  -v redis_data:/data \
  redis:7-alpine \
  redis-server --requirepass tradingagents123
```

#### 数据库初始化

```bash
# 运行初始化脚本
mongo < scripts/mongo-init.js

# 或使用Python脚本
python -m scripts.init_database
```

### 4.3 后端部署

#### 1. 配置环境变量

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑.env文件
nano .env
```

关键配置项：

```env
# 数据库连接（本地部署）
MONGODB_HOST=localhost
MONGODB_PORT=27017
MONGODB_USERNAME=tradingagents
MONGODB_PASSWORD=your-password
MONGODB_DATABASE=tradingagentscn

REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=tradingagents123

# LLM配置
DEEPSEEK_API_KEY=your-api-key
DEEPSEEK_ENABLED=true

# 数据源
DEFAULT_CHINA_DATA_SOURCE=akshare
```

#### 2. 启动后端服务

```bash
# 开发模式（自动重载）
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 生产模式
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4

# 或使用启动命令
python -m app
```

#### 3. 验证后端

```bash
# 测试API
curl http://localhost:8000/api/health

# 查看API文档
# 浏览器访问: http://localhost:8000/docs
```

### 4.4 前端部署

#### 1. 安装Node.js依赖

```bash
cd frontend

# 使用npm
npm install

# 或使用yarn
yarn install
```

#### 2. 配置环境变量

创建 `frontend/.env.production`:

```env
# API地址
VITE_API_BASE_URL=http://localhost:8000

# 其他配置
VITE_APP_TITLE=TradingAgents-CN
```

#### 3. 构建前端

```bash
# 开发模式
npm run dev

# 生产构建
npm run build

# 预览构建结果
npm run preview
```

#### 4. 使用Nginx部署

```bash
# 安装Nginx
sudo apt-get install nginx

# 复制构建产物
sudo cp -r dist/* /var/www/html/

# 配置Nginx
sudo nano /etc/nginx/sites-available/tradingagents
```

Nginx配置示例：

```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 5. 配置说明

### 5.1 环境变量详解

#### 必需配置

| 变量名 | 说明 | 示例值 | 默认值 |
|--------|------|--------|--------|
| `MONGODB_HOST` | MongoDB主机地址 | localhost | localhost |
| `MONGODB_PORT` | MongoDB端口 | 27017 | 27017 |
| `MONGODB_USERNAME` | MongoDB用户名 | tradingagents | - |
| `MONGODB_PASSWORD` | MongoDB密码 | your-password | - |
| `MONGODB_DATABASE` | 数据库名称 | tradingagentscn | tradingagents |
| `REDIS_HOST` | Redis主机地址 | localhost | localhost |
| `REDIS_PORT` | Redis端口 | 6379 | 6379 |
| `JWT_SECRET` | JWT密钥 | random-string | - |
| `CSRF_SECRET` | CSRF密钥 | random-string | - |

#### LLM配置

| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `DEEPSEEK_API_KEY` | DeepSeek API密钥 | sk-xxxxx |
| `DEEPSEEK_BASE_URL` | DeepSeek API地址 | https://api.deepseek.com |
| `DASHSCOPE_API_KEY` | 阿里百炼密钥 | sk-xxxxx |
| `AIHUBMIX_API_KEY` | AIHubMix密钥 | sk-xxxxx |
| `OPENAI_API_KEY` | OpenAI密钥 | sk-xxxxx |
| `GOOGLE_API_KEY` | Google AI密钥 | AIzaxxxxx |

#### 数据源配置

| 变量名 | 说明 | 可选值 |
|--------|------|--------|
| `DEFAULT_CHINA_DATA_SOURCE` | 默认A股数据源 | akshare, tushare, baostock |
| `TUSHARE_TOKEN` | Tushare Token | - |
| `FINNHUB_API_KEY` | FinnHub密钥 | - |

### 5.2 数据库连接配置

#### MongoDB连接字符串格式

```
mongodb://[username:password@]host[:port][/database][?options]
```

示例：

```env
# 标准连接
MONGODB_URL=mongodb://tradingagents:password@localhost:27017/tradingagentscn

# Docker内部连接
MONGODB_URL=mongodb://admin:tradingagents123@mongodb:27017/tradingagents?authSource=admin

# 副本集连接
MONGODB_URL=mongodb://user:pass@host1:27017,host2:27017/db?replicaSet=myReplicaSet
```

#### Redis连接配置

```env
# 标准连接
REDIS_URL=redis://:password@localhost:6379/0

# 无密码连接
REDIS_URL=redis://localhost:6379/0

# Docker内部连接
REDIS_URL=redis://:tradingagents123@redis:6379/0
```

### 5.3 日志配置

```env
# 日志级别
TRADINGAGENTS_LOG_LEVEL=INFO

# 日志目录
TRADINGAGENTS_LOG_DIR=./logs

# 日志文件
TRADINGAGENTS_LOG_FILE=tradingagents.log

# 日志轮转
LOG_MAX_BYTES=100MB
LOG_BACKUP_COUNT=10
```

---

## 6. 验证部署

### 6.1 健康检查

```bash
# 后端健康检查
curl http://localhost:8000/api/health

# 前端健康检查
curl http://localhost:3000/health

# 数据库连接检查
docker exec -it tradingagents-mongodb mongo --eval "db.adminCommand('ping')"

# Redis连接检查
docker exec -it tradingagents-redis redis-cli ping
```

### 6.2 功能测试

#### 1. 测试API接口

```bash
# 获取系统配置
curl http://localhost:8000/api/config/system

# 测试登录
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"your-password"}'
```

#### 2. 测试数据同步

```bash
# 同步单只股票
curl -X POST http://localhost:8000/api/sync/stock \
  -H "Content-Type: application/json" \
  -d '{"symbol":"000001","market_type":"CN"}'

# 查看同步状态
curl http://localhost:8000/api/sync/status
```

#### 3. 测试分析功能

在Web界面中：
1. 注册/登录账号
2. 进入"股票分析"页面
3. 输入股票代码（如：000001）
4. 选择LLM模型
5. 点击"开始分析"
6. 观察分析进度和结果

### 6.3 性能基准

```bash
# 使用ab工具测试
ab -n 100 -c 10 http://localhost:8000/api/health

# 查看资源使用
docker stats
```

---

## 7. 数据同步

### 7.1 为什么需要数据同步？

在分析股票之前，必须先同步股票数据，包括：
- 基础信息（股票名称、行业、板块）
- 历史行情（K线数据）
- 财务数据（财务报表）
- 实时行情（最新价格）

### 7.2 批量数据同步

#### 通过Web界面

1. 登录系统
2. 进入"数据管理" → "数据同步"
3. 选择同步类型：
   - **基础信息同步**: 所有A股基本信息
   - **历史数据同步**: 指定日期范围的历史行情
   - **财务数据同步**: 财务报表数据
4. 点击"开始同步"

#### 通过API

```bash
# 同步所有A股基础信息
curl -X POST http://localhost:8000/api/sync/basics \
  -H "Authorization: Bearer your-token"

# 同步单只股票
curl -X POST http://localhost:8000/api/sync/stock \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-token" \
  -d '{"symbol":"000001","market_type":"CN"}'
```

#### 通过CLI

```bash
# 同步所有A股基础信息
python -m cli sync-basics

# 同步单只股票
python -m cli sync-stock 000001

# 批量同步
python -m cli sync-batch --symbols 000001,000002,600000
```

### 7.3 数据源优先级

系统会自动按照以下顺序尝试获取数据：

1. **AkShare** (免费，推荐)
2. **Tushare** (需要Token，数据质量高)
3. **BaoStock** (免费，有限制)

如果第一个数据源失败，会自动切换到下一个。

---

## 8. 常见问题

### 8.1 Docker相关问题

#### Q1: 容器启动失败

```bash
# 查看详细日志
docker-compose logs backend

# 常见原因：
# 1. 端口被占用 → 修改docker-compose.yml中的端口映射
# 2. 内存不足 → 增加Docker内存限制
# 3. 环境变量错误 → 检查.env文件配置
```

#### Q2: 数据库连接失败

```bash
# 检查数据库容器状态
docker-compose ps mongodb
docker-compose logs mongodb

# 确认数据库健康检查通过
docker exec tradingagents-mongodb mongo --eval "db.adminCommand('ping')"

# 检查网络连接
docker exec tradingagents-backend ping mongodb
```

#### Q3: 前端无法访问后端API

```bash
# 检查CORS配置
# 在.env中设置正确的CORS_ORIGINS
CORS_ORIGINS=http://localhost:3000,http://localhost:8080

# 检查前端API配置
# frontend/.env.production
VITE_API_BASE_URL=http://localhost:8000
```

### 8.2 功能相关问题

#### Q4: 分析失败或超时

可能原因和解决方案：

1. **LLM API密钥无效**
   ```bash
   # 验证API密钥
   curl https://api.deepseek.com/v1/models \
     -H "Authorization: Bearer your-api-key"
   ```

2. **数据未同步**
   - 先执行数据同步，确保股票数据完整

3. **并发限制过高**
   - 降低并发数：系统设置 → 最大并发任务数

4. **网络问题**
   - 检查代理设置（如果使用国外LLM）
   - 确保网络连通性

#### Q5: 数据同步失败

```bash
# 查看同步日志
docker-compose logs backend | grep sync

# 常见原因：
# 1. Tushare Token无效 → 更新Token
# 2. API限流 → 减少并发数
# 3. 网络问题 → 检查网络连接
```

#### Q6: 报告导出失败

```bash
# 检查依赖工具
docker exec tradingagents-backend which pandoc
docker exec tradingagents-backend which wkhtmltopdf

# 如果缺失，需要重新构建镜像
docker-compose build backend --no-cache
```

### 8.3 性能问题

#### Q7: 系统响应缓慢

优化建议：

1. **启用Redis缓存**
   ```env
   TRADINGAGENTS_CACHE_TYPE=redis
   ```

2. **调整数据库连接池**
   ```python
   # 在config中调整
   MONGODB_MAX_POOL_SIZE=50
   MONGODB_MIN_POOL_SIZE=10
   ```

3. **增加Worker数量**
   ```bash
   # 修改启动命令
   uvicorn app.main:app --workers 4
   ```

#### Q8: 内存占用过高

```bash
# 限制容器内存
docker-compose up -d --scale backend=1

# 或在docker-compose.yml中设置
services:
  backend:
    deploy:
      resources:
        limits:
          memory: 4G
```

---

## 9. 性能优化

### 9.1 数据库优化

#### MongoDB索引优化

```javascript
// 创建复合索引
db.analysis_reports.createIndex({
  symbol: 1,
  created_at: -1,
  user_id: 1
})

// 查看索引使用情况
db.analysis_reports.getIndexes()
```

#### 查询优化

```python
# 使用投影减少数据传输
db.collection.find({}, {"field1": 1, "field2": 1})

# 使用limit限制结果数量
db.collection.find().limit(100)
```

### 9.2 缓存策略

#### Redis缓存配置

```python
# 启用多级缓存
CACHE_CONFIG = {
    "default_ttl": 3600,      # 默认缓存1小时
    "stock_info_ttl": 86400,  # 股票信息缓存24小时
    "quote_ttl": 60,          # 行情缓存1分钟
}
```

#### 缓存预热

```bash
# 启动时预热常用数据
python -m scripts.warmup_cache
```

### 9.3 并发优化

#### 异步处理

```python
# 使用异步任务队列
from app.services.queue_service import QueueService

queue = QueueService()
queue.enqueue_analysis_task(symbol="000001")
```

#### 连接池配置

```python
# MongoDB连接池
MONGODB_CONFIG = {
    "maxPoolSize": 50,
    "minPoolSize": 10,
    "maxIdleTimeMS": 30000,
}
```

---

## 10. 监控与维护

### 10.1 日志管理

#### 查看日志

```bash
# Docker日志
docker-compose logs -f backend
docker-compose logs backend --tail 100

# 应用日志
tail -f logs/tradingagents.log

# 日志级别
TRADINGAGENTS_LOG_LEVEL=DEBUG
```

#### 日志分析

```bash
# 统计错误数量
grep "ERROR" logs/tradingagents.log | wc -l

# 查看最近1小时的错误
grep "ERROR" logs/tradingagents.log | tail -100
```

### 10.2 数据备份

#### MongoDB备份

```bash
# 备份
docker exec tradingagents-mongodb mongodump \
  --username=admin \
  --password=tradingagents123 \
  --db=tradingagentscn \
  --out=/backup/

# 恢复
docker exec tradingagents-mongodb mongorestore \
  --username=admin \
  --password=tradingagents123 \
  --db=tradingagentscn \
  /backup/
```

#### Redis备份

```bash
# 备份RDB文件
docker cp tradingagents-redis:/data/dump.rdb ./backup/

# 恢复
docker cp ./backup/dump.rdb tradingagents-redis:/data/dump.rdb
```

### 10.3 系统监控

#### 容器监控

```bash
# 查看容器资源使用
docker stats

# 查看特定容器
docker stats tradingagents-backend tradingagents-mongodb
```

#### 应用监控

```bash
# 查看API响应时间
curl -w "@curl-format.txt" http://localhost:8000/api/health

# curl-format.txt内容：
# time_namelookup: %{time_namelookup}\n
# time_connect: %{time_connect}\n
# time_appconnect: %{time_appconnect}\n
# time_pretransfer: %{time_pretransfer}\n
# time_starttransfer: %{time_starttransfer}\n
# time_total: %{time_total}\n
```

### 10.4 定期维护

#### 数据清理

```bash
# 清理过期日志
find logs/ -name "*.log" -mtime +30 -delete

# 清理过期分析结果
python -m scripts.cleanup_old_data --days 90
```

#### 数据库维护

```javascript
// MongoDB
db.collection.reIndex()
db.runCommand({compact: "collection_name"})

// Redis
redis-cli --latency
redis-cli --bigkeys
```

#### 系统更新

```bash
# 拉取最新代码
git pull origin main

# 更新依赖
pip install -r requirements.txt --upgrade

# 重新构建镜像
docker-compose build

# 重启服务
docker-compose up -d
```

---

## 附录

### A. 端口映射表

| 服务 | 内部端口 | 外部端口 | 说明 |
|------|---------|---------|------|
| Backend | 8000 | 8000 | FastAPI服务 |
| Frontend | 80 | 3000 | Nginx前端 |
| MongoDB | 27017 | 27017 | 数据库 |
| Redis | 6379 | 6379 | 缓存 |
| Mongo Express | 8081 | 8082 | 管理界面（可选） |
| Redis Commander | 8081 | 8081 | 管理界面（可选） |

### B. 目录结构

```
TradingAgents-CN/
├── app/                    # 后端代码
├── frontend/               # 前端代码
├── tradingagents/          # 核心分析系统
├── cli/                    # CLI工具
├── scripts/                # 工具脚本
├── docs/                   # 文档
├── config/                 # 配置文件
├── logs/                   # 日志文件
├── data/                   # 数据文件
├── docker-compose.yml      # Docker编排配置
├── Dockerfile.backend      # 后端镜像配置
├── Dockerfile.frontend     # 前端镜像配置
├── .env.example            # 环境变量模板
└── .env                    # 实际环境变量（需创建）
```

### C. 有用的命令

```bash
# Docker相关
docker-compose up -d              # 启动所有服务
docker-compose down               # 停止所有服务
docker-compose logs -f [service]  # 查看日志
docker-compose exec [service] sh # 进入容器
docker-compose restart [service]  # 重启服务

# 数据库相关
docker exec -it tradingagents-mongodb mongo  # 进入MongoDB
docker exec -it tradingagents-redis redis-cli # 进入Redis

# 应用相关
python -m cli                    # 启动CLI
python -m app                    # 启动后端
python tests/test_xxx.py         # 运行测试
```

### D. 故障排除流程图

```
问题发生
  │
  ├─→ 检查服务状态
  │   ├─ docker-compose ps
  │   └─ systemctl status
  │
  ├─→ 查看日志
  │   ├─ docker-compose logs
  │   └─ tail -f logs/*.log
  │
  ├─→ 检查配置
  │   ├─ 验证.env文件
  │   └─ 检查环境变量
  │
  ├─→ 测试连接
  │   ├─ ping/curl测试
  │   └─ 数据库连接测试
  │
  └─→ 查阅文档
      ├─ README.md
      ├─ docs/
      └─ GitHub Issues
```

---

## 获取帮助

- **文档**: https://github.com/hsliuping/TradingAgents-CN/tree/main/docs
- **问题反馈**: https://github.com/hsliuping/TradingAgents-CN/issues
- **微信公众号**: TradingAgents-CN
- **邮箱**: hsliup@163.com

---

**祝您部署顺利！**

*本文档最后更新: 2026-05-13*
