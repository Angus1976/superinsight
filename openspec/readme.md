# SuperInsight 企业级数据洞察平台

基于 Apache Superset 深度定制的企业级数据洞察与知识治理平台。  
让业务人员通过图表标注快速释放数据价值，技术人员可按量计费结算，支持 SaaS、私有化、混合部署。

## 核心亮点
- 完整继承 Superset 所有 BI 能力（仪表盘、图表、SQL Lab、警报、嵌入等）
- 图表实时协作标注（文字、箭头、矩形、高亮），支持业务/技术双模式
- 标注即生成治理后知识语料 + 业务规则自动注入与校验
- 业务规则库上下文感知推荐 + Warning 校验
- 标注工时自动计时、按量结算、月度账单 + 客户对账报表
- 语料版本快照 + 批量回滚
- 混合部署（数据永不落云）+ 连接状态监控 + Redis 热缓存
- MongoDB 分片、历史归档、大宽表瓦片化
- 细粒度权限 + 数据脱敏 + 角色矩阵
- 管理员后台（租户管理、账单、监控）

## 快速开始

### 私有化部署（推荐本地测试）
```bash
# 1. 克隆项目
git clone https://github.com/your-org/superinsight.git
cd superinsight

# 2. 复制环境变量
cp .env.example .env
# 编辑 .env 文件，填写 MySQL、Redis、MongoDB 等密码

# 3. 启动（首次会自动拉取镜像）
docker-compose up -d

# 4. 访问
http://localhost:8088
默认账号：admin / admin
SaaS 版（腾讯云一键部署）

将自定义镜像推送到腾讯云 TCR
在云托管创建服务，选择 TCR 镜像
配置环境变量（TENANT_MODE=multi 等）
绑定域名（支持子域名如 tenant1.superinsight.cn）

混合部署（数据留在客户内网）

应用层部署在腾讯云
数据源通过 Frp / ZeroTier / 专线直连
后台可查看“本地连接状态”监控面板

项目结构
textsuperinsight/
├── backend/                # 后端代码（Superset 定制 + 自定义模块）
├── frontend/               # 前端 React 组件（标注工具、规则库、账单等）
├── openspec/               # 所有模块详细设计文档（AI 辅助开发用）
├── docs/                   # PRD、部署环境等
├── scripts/                # 批处理脚本（月结、归档等）
├── docker-compose.yml      # 私有化部署
├── docker-compose.saas.yml # SaaS 版参考
├── .env.example
└── README.md
文档

产品需求文档（PRD）
部署环境说明
所有模块详细设计（推荐按顺序交给 AI 生成代码）

技术栈

后端：Python 3.11 + Flask + Apache Superset 4.0.2+
数据库：MySQL 8.0 + MongoDB 7.x + Redis 7.x
前端：React + Konva.js（标注画布）+ Ant Design
部署：Docker Compose / 腾讯云 TCB / 云托管

许可证
MIT License
联系与合作
如需商业合作、定制开发、私有化部署服务，或有任何问题，欢迎联系：
邮箱：xxx@email.com
微信/企业微信：（可补充）
最后更新：2025年12月10日
