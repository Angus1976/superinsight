# 后端核心规范

- 支持：多租户隔离、中文优先、定制 Superset 配置
- 包括：统一认证、服务编排、中文搜索/分词优化、中间件扩展点
- 质量：覆盖高可用性测试、容器化部署脚本与监控埋点

# 模块：backend-core（后端核心）

## 目标
- Superset 基础定制
- 多租户隔离（子域名/路径）
- 强制中文 + 业务化术语
- 核心配置（superset_config.py）

## 交付物
- superset_config.py（完整多租户配置）
- custom/security/ 目录（TenantMixin、权限过滤器）
- translations/zh.json（补充 200+ 业务术语）

## 关键实现点
- tenant_id 上下文（request.headers.get('X-Tenant-ID')）
- 所有模型自动添加 tenant_id 过滤
- 强制语言 zh_CN

# 模块：backend-core（后端核心）

## 目标
- 基于 Apache Superset 4.0.2+ 进行基础定制
- 实现多租户隔离（支持子域名或路径区分租户，如 tenant1.superinsight.cn）
- 强制默认中文界面 + 自定义业务化术语（e.g., “Slice” → “图表”, “Database” → “数据源”）
- 配置 Superset 核心设置（缓存、Celery、环境变量）
- 支持混合部署的 tenant_id 上下文注入

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化目录）

## 交付物
1. **superset_config.py**：完整配置文件（多租户、缓存、中文设置）
2. **custom/security/manager.py**：多租户权限管理器重写（TenantMixin + 过滤器）
3. **custom/security/filters.py**：所有查询自动添加 tenant_id 过滤
4. **translations/zh.json**：自定义翻译文件（至少 200+ 业务术语补充）
5. **docker/Dockerfile**：自定义 Superset 镜像构建文件
6. **docker/Dockerfile.worker**：Celery Worker 镜像
7. **docker/entrypoint.sh**：启动脚本（初始化多租户）

## 关键实现点
### 1. 多租户隔离
- 在 Flask request 钩子中注入 tenant_id：
  ```python
  @app.before_request
  def inject_tenant():
      g.tenant_id = request.headers.get('X-Tenant-ID') or request.path.split('/tenant/')[1] if '/tenant/' in request.path else None
      if not g.tenant_id:
          abort(403, "Missing Tenant ID")

所有模型继承 TenantMixin：Pythonclass TenantMixin:
    tenant_id = Column(String(64), nullable=False)
数据库连接改造：添加 ?tenant_id=xxx 参数

2. 中文界面 + 术语优化

superset_config.py 中设置：PythonLANGUAGES = {'zh': {'flag': 'cn', 'name': 'Chinese'}}
DEFAULT_LANGUAGE = 'zh'
zh.json 示例：JSON{
  "SLICE": "图表",
  "DATABASE": "数据源",
  "DASHBOARD": "仪表盘"
  // 补充 200+ 术语...
}

3. Celery + Redis 配置

superset_config.py：PythonCELERY_CONFIG = {
    'broker_url': 'redis://redis:6379/0',
    'result_backend': 'redis://redis:6379/0'
}

4. 混合部署支持

支持环境变量 TENANT_MODE = 'multi' 或 'single'

测试说明

运行 docker-compose up -d
访问 http://localhost:8088，检查是否默认中文
发送请求带 X-Tenant-ID: test，验证隔离

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 backend-core 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"

