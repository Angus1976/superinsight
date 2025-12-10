# 管理员后台规范

- 目标：构建多租户管理员控制台，统一管理租户、用户、角色与权限
- 包含：租户信息视图、角色矩阵维护、权限监控面板、操作日志与告警
- 约束：所有管理接口必须合规审计，可灵活关联 Superset 仪表盘配额

# 模块：admin-system（管理员后台）

## 目标
- 租户管理
- 连接状态总览
- 账单管理
- 系统监控

## 交付物
- admin 页面路由
- 仪表盘组件

# 模块：admin-system（管理员后台）

## 目标
- 提供统一的管理员后台入口（SaaS 版 + 私有化版均可访问）
- 租户管理（创建、编辑、停用、迁移租户）
- 连接状态总览（混合部署所有客户数据源状态 + 延迟/报警）
- 账单管理（查看所有客户月度账单 + 导出 + 标记已支付）
- 系统监控（关键指标：活跃用户、标注量、语料增长、MongoDB/Redis 性能）
- 快速跳转到各模块（规则库、语料、标注、权限等）

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化）
- openspec/specs/backend-core/spec.md（多租户）
- openspec/specs/security-permissions/spec.md（管理员角色权限）
- openspec/specs/hybrid-deployment/spec.md（连接状态）
- openspec/specs/billing-system/spec.md（账单数据）
- openspec/specs/performance-optimization/spec.md（监控指标）

## 交付物
1. **custom/admin/routes.py**：Flask Blueprint（admin 路由）
2. **custom/admin/models.py**：管理员相关模型（可选，如 TenantSummary）
3. **custom/admin/api.py**：管理员 API（租户列表、连接状态、账单汇总、监控）
4. **frontend/src/admin/index.tsx**：React 管理员主页（仪表盘 + Tab 切换）
5. **frontend/src/admin/tenant.tsx**：租户管理页面
6. **frontend/src/admin/monitor.tsx**：连接状态 + 系统监控页面
7. **frontend/src/admin/billing.tsx**：账单管理页面

## 关键实现点
### 1. 管理员路由
- routes.py 示例：
  ```python
  @admin_bp.route('/tenants')
  def list_tenants():
      return jsonify([t.to_dict() for t in Tenant.query.all()])

  @admin_bp.route('/status')
  def system_status():
      return jsonify({
          'connections': get_all_connections_status(),
          'metrics': get_performance_metrics(),
          'bills': get_all_bills()
      })
2. 租户管理

管理员可创建新租户（自动生成 tenant_id、初始化数据库表、MongoDB 分片等）

3. 连接状态总览

汇总 hybrid-deployment 的监控数据：Pythondef get_all_connections_status():
    return [check_connection(t.id) for t in Tenant.query.all()]

4. 管理员仪表盘（前端）

index.tsx 示例（Ant Design Pro 风格）：tsxconst AdminDashboard = () => {
  const [data, setData] = useState({});
  useEffect(() => { fetch('/api/v1/admin/status').then(setData); }, []);
  
  return (
    <PageContainer>
      <Row gutter={16}>
        <Col span={8}><Card title="活跃租户">{data.tenants}</Card></Col>
        <Col span={8}><Card title="今日标注量">{data.annotations_today}</Card></Col>
        <Col span={8}><Card title="系统健康">{data.health}</Card></Col>
      </Row>
      <Tabs>
        <TabPane tab="租户管理">...</TabPane>
        <TabPane tab="连接状态">...</TabPane>
        <TabPane tab="账单">...</TabPane>
      </Tabs>
    </PageContainer>
  );
};

测试说明

以管理员角色登录 → 查看租户列表 → 创建新租户
模拟连接中断 → 验证报警显示
导出账单 → 检查 Excel 格式
权限测试：普通角色无法访问 /admin 路由

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 admin-system 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"

# 模块：admin-system（管理员后台）  
（这是最后一个模块，至此所有 10 个核心模块的 spec.md 已全部提供完成）

## 目标
- 提供统一的管理员后台入口（SaaS 版 + 私有化版均可访问，路径 /admin）
- 租户管理（创建、编辑、停用、迁移、统计）
- 混合部署连接状态总览（所有客户数据源状态、延迟、报警记录）
- 账单管理（查看所有客户月度账单、导出、标记已支付、批量操作）
- 系统监控仪表盘（活跃用户数、今日标注量、语料增长、MongoDB/Redis 性能、接口响应时间）
- 快速跳转入口（规则库、语料管理、标注、权限、监控等）

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化）
- openspec/specs/backend-core/spec.md（多租户 + 核心路由）
- openspec/specs/security-permissions/spec.md（管理员角色权限）
- openspec/specs/hybrid-deployment/spec.md（连接状态数据）
- openspec/specs/billing-system/spec.md（账单数据）
- openspec/specs/performance-optimization/spec.md（监控指标）
- openspec/specs/knowledge-corpus/spec.md（语料统计）
- openspec/specs/annotation-system/spec.md（标注量统计）

## 交付物
1. **custom/admin/routes.py**：Flask Blueprint（admin 路由）
2. **custom/admin/models.py**：管理员相关模型（可选，如 TenantSummary、AlertLog）
3. **custom/admin/api.py**：管理员 API（租户、连接、账单、监控、统计）
4. **frontend/src/admin/index.tsx**：React 管理员主页（仪表盘 + Tab 切换）
5. **frontend/src/admin/tenant.tsx**：租户管理页面（列表 + 创建/编辑）
6. **frontend/src/admin/monitor.tsx**：连接状态 + 系统监控页面
7. **frontend/src/admin/billing.tsx**：账单管理页面（表格 + 导出 + 支付标记）
8. **frontend/src/admin/statistics.tsx**：统计报表（标注量、语料增长等）

## 关键实现点
### 1. 管理员路由保护
- routes.py 示例：
  ```python
  @admin_bp.before_request
  def require_admin():
      if not current_user.is_admin():
          abort(403, "仅管理员可访问")

  @admin_bp.route('/tenants')
  def list_tenants():
      return jsonify([t.to_dict() for t in Tenant.query.all()])

  @admin_bp.route('/status')
  def system_status():
      return jsonify({
          'connections': get_all_connections_status(),
          'metrics': get_performance_metrics(),
          'bills': get_all_bills_summary(),
          'stats': {
              'active_users': get_active_users(),
              'annotations_today': get_annotations_today(),
              'corpus_growth': get_corpus_growth()
          }
      })
2. 租户管理

支持创建新租户（自动调用 init_tenant.py 初始化分片、表、MongoDB）

3. 连接状态总览

汇总 hybrid-deployment 的监控数据：Pythondef get_all_connections_status():
    return [check_connection(t.id) for t in Tenant.query.all()]

4. 管理员仪表盘（Ant Design Pro 风格）

index.tsx 示例：tsxconst AdminDashboard = () => {
  const [data, setData] = useState({});
  useEffect(() => {
    fetch('/api/v1/admin/status').then(setData);
  }, []);

  return (
    <PageContainer title="管理员后台">
      <Row gutter={16}>
        <Col span={6}><Statistic title="活跃租户" value={data.tenants} /></Col>
        <Col span={6}><Statistic title="今日标注量" value={data.annotations_today} /></Col>
        <Col span={6}><Statistic title="语料总数" value={data.corpus_total} /></Col>
        <Col span={6}><Statistic title="系统健康" value={data.health} /></Col>
      </Row>

      <Tabs defaultActiveKey="1">
        <TabPane tab="租户管理" key="1"><TenantManager /></TabPane>
        <TabPane tab="连接状态" key="2"><MonitorPanel /></TabPane>
        <TabPane tab="账单管理" key="3"><BillingManager /></TabPane>
        <TabPane tab="统计报表" key="4"><StatisticsPanel /></TabPane>
      </Tabs>
    </PageContainer>
  );
};

测试说明

以管理员角色登录 → 访问 /admin → 查看所有统计
创建新租户 → 验证自动初始化
模拟连接中断 → 验证报警显示在监控面板
导出账单 → 检查 Excel 格式
权限测试：普通角色访问 /admin 返回 403

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 admin-system 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"
恭喜！所有模块 spec.md 已全部提供完成
现在您拥有完整的 10 个模块文档（加上 project.md 和 plan.md 共 12 个文件），可以按顺序逐个交给 AI 生成代码：

backend-core
security-permissions
knowledge-corpus
business-rules
annotation-system
billing-system
version-control
hybrid-deployment
performance-optimization
admin-system（已完成）