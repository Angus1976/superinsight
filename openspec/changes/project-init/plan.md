# 项目启动计划

目标时间：3 天迭代 0

1. 确定总体架构与模块划分，明确每个 Agent 责任
2. 初步梳理管理员后台、协作标注、知识语料、业务规则等核心系统的关键流程
3. 输出各模块的接口需求、依赖关系和验收标准

# 项目启动计划（切块 1）

## 1. 初始化目录结构
创建 openspec/ 目录及所有子文件夹（见 project.md）

## 2. 基础文件
- README.md
- docker-compose.yml（私有化）
- docker-compose.saas.yml（腾讯云托管）
- .env.example
- requirements.txt

## 3. 开发顺序建议
按 specs/ 目录顺序逐个完成，每个 spec.md 完成后立即 commit

## 4. 交付标准
每个模块完成后提供：
- 完整代码文件列表（路径 + 文件名）
- 每个文件的完整代码
- 运行/测试说明

# SuperInsight 项目启动计划

## 1. 项目初始化步骤
### 步骤 1: 创建 Git 仓库
- 创建新仓库：superinsight-1.3
- 初始化：git init
- 添加远程：git remote add origin [您的仓库 URL]

### 步骤 2: 设置环境
- 安装 Python 3.11
- 安装 Node.js 20+
- 安装 Docker & Docker Compose
- 腾讯云账号准备（TCB / TCR / 云托管）

### 步骤 3: 基础目录结构创建
创建以下目录树（参考 openspec/project.md）：

superinsight-1.3/
├── backend/
│   ├── custom/              # 所有自定义模块
│   │   ├── security/
│   │   ├── annotations/
│   │   ├── billing/
│   │   ├── rls/
│   │   ├── knowledge/
│   │   ├── rules/
│   │   ├── version/
│   │   ├── performance/
│   │   ├── hybrid/
│   │   └── websocket.py
│   ├── docker/
│   │   ├── Dockerfile
│   │   ├── Dockerfile.worker
│   │   └── entrypoint.sh
│   ├── migrations/          # Alembic 迁移
│   └── superset_config.py
├── frontend/
│   └── superset-frontend/
│       ├── packages/superset-ui-core/translations/zh.json
│       └── src/
│           ├── components/AnnotationTool/
│           ├── dashboard/
│           ├── billing/
│           ├── knowledge/
│           ├── rules/
│           ├── version/
│           └── hybrid/
├── scripts/
│   ├── init_tenant.py
│   ├── generate_monthly_bill.py
│   └── archive_old_corpus.py
├── openspec/                # 文档区（当前文件所在）
│   ├── project.md
│   ├── changes/
│   │   └── project-init/
│   │       └── plan.md      # 本文件
│   └── specs/               # 所有模块 spec.md
├── docker-compose.yml
├── docker-compose.saas.yml
├── .env.example
├── requirements.txt
├── package.json
└── README.md


### 步骤 4: 基础文件生成
- **README.md**：
  项目概述、安装指南、运行命令。
- **docker-compose.yml**（私有化版示例）：
```yaml
version: '3.8'
services:
  superset:
    image: apache/superset:4.0.2
    ports: ["8088:8088"]
    depends_on: [redis, mysql, mongodb]
    volumes: ["./backend:/app/pythonpath"]
    environment:
      - SUPERSET_ENV=production
      - TENANT_MODE=multi
  redis:
    image: redis:7
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
  mongodb:
    image: mongo:7

    docker-compose.saas.yml（腾讯云版示例）：
类似，但使用 TCR 镜像 + 云托管环境变量。
.env.example：

textMYSQL_ROOT_PASSWORD=root
REDIS_HOST=redis
MONGODB_URI=mongodb://mongodb:27017/superinsight
SECRET_KEY=your_secret

requirements.txt：

textapache-superset==4.0.2
flask-socketio
pymongo
mysqlclient
redis
celery
konva  # 对于前端，但后端无需
步骤 5: 开发流程

按 openspec/project.md 中的顺序开发每个 specs/ 模块
每个模块完成后：运行 docker-compose up 测试
使用 AI 逐个生成代码：把 spec.md 作为 Prompt 输入
集成测试：每3个模块后运行全系统

步骤 6: 部署指南

私有化：docker-compose up -d
SaaS：推送镜像到 TCR → 云托管创建服务

步骤 7: 时间预估
总开发 55-65 天（3-5人团队），每个模块 5-10 天
本计划完成后，立即开始 specs/backend-core/spec.md
