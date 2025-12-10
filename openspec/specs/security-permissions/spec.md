# 权限体系规范

- 范围：定义角色矩阵、最小权限、脱敏策略与审计链
- 特性：上下级角色继承、敏感字段自动脱敏、行为审批+回溯记录
- 接口：暴露权限分页查询、角色组合验证与异常警报接口

# 模块：security-permissions（权限体系）

## 目标
- 四层权限 + 语料权限
- 角色矩阵实现
- 数据脱敏（外包角色）

## 交付物
- custom/security/ 扩展
- 脱敏过滤器
- 权限中间件

# 模块：security-permissions（权限体系）

## 目标
- 实现四层灵活权限体系：租户级 → 数据源分类级 → 数据集/语料集级 → 行级安全 (RLS)
- 支持知识语料权限（治理前/后语料的读/写/审核）
- 内置角色矩阵（管理员、业务标注员、技术开发者、只读访客、外包标注人员）
- 数据脱敏支持（e.g., 外包角色仅见模糊数据，如金额显示为 ***）
- 与 Superset 原生权限无缝整合（重写 SecurityManager）

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化）
- openspec/specs/backend-core/spec.md（多租户 tenant_id）

## 交付物
1. **custom/security/manager.py**：重写 Superset SecurityManager，注入 tenant_id + 角色矩阵校验
2. **custom/security/permissions.py**：权限定义 + 四层过滤逻辑
3. **custom/security/desensitize.py**：数据脱敏函数（e.g., mask sensitive fields）
4. **custom/security/matrix.json** 或 **matrix.py**：角色权限矩阵配置（JSON 或 dict）
5. **custom/security/middleware.py**：Flask 中间件（API 权限拦截）

## 关键实现点
### 1. 四层权限体系
- 租户级：所有查询自动 filter tenant_id == g.tenant_id
- 数据源分类级：使用 superset_datasource_category 表关联权限
- 数据集/语料集级：superset_datasource_access 表多维度（部门/岗位/个人）
- 行级安全 (RLS)：动态生成 View，支持 MySQL/ClickHouse
  示例 RLS：
  ```python
  def apply_rls(query, user):
      if user.role == 'business':
          query = query.filter('sales > 0')  # 只见正值
      return query
2. 角色矩阵实现

定义矩阵（matrix.py 示例）：PythonROLE_MATRIX = {
    'admin': {'corpus_pre': 'CRUD', 'corpus_post': 'CRUD', 'rules': 'CRUD', 'annotate': 'full', 'rollback': 'full'},
    'business_annotator': {'corpus_pre': 'R (desensitized)', 'corpus_post': 'CRU', 'rules': 'R', 'annotate': 'business', 'rollback': 'R'},
    'tech_developer': {'corpus_pre': 'CRUD', 'corpus_post': 'CRUD', 'rules': 'CRUD', 'annotate': 'tech', 'rollback': 'full'},
    'viewer': {'corpus_pre': 'none', 'corpus_post': 'R', 'rules': 'R', 'annotate': 'none', 'rollback': 'none'},
    'outsourcer': {'corpus_pre': 'R (desensitized)', 'corpus_post': 'CRU', 'rules': 'R', 'annotate': 'business', 'rollback': 'none'}
}
在 manager.py 中校验：get_permissions(user.role, entity_type)

3. 数据脱敏

desensitize.py 示例：Pythondef desensitize_data(data, role):
    if role == 'outsourcer':
        data['amount'] = '***'  # 模糊金额
    return data
在查询后钩子应用：post_query_hook(desensitize_data)

4. 语料权限整合

entity_type = 'corpus' 时，使用 knowledge_corpus 表 tenant_id + category_id 过滤

测试说明

创建测试用户（不同角色）
查询语料：验证业务角色只能见治理后语料，外包见脱敏数据
尝试越权操作：应返回 403

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 security-permissions 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"