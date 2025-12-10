# 业务规则库规范

- 功能：支持规则 CRUD、版本管理、智能推荐与自动校验提示
- 推荐策略：根据使用频率与命中率提示可复用规则片段
- 校验链路：规则变更需经过审批，执行时提供详细命中路径
# 模块：business-rules（业务规则库）

## 目标
- 规则 CRUD
- 上下文感知推荐（根据图表 Metric/Dimension）
- 自动校验反馈（Warning）

## 交付物
- custom/rules/ 目录
- recommend_rules API
- validate_data_against_rule 函数

# 模块：business-rules（业务规则库）

## 目标
- 支持业务规则的创建、读取、更新、删除 (CRUD)
- 规则定义包括类型（阈值、模式、分类等）、表达式（e.g., "sales > 10000 ? 'high' : 'low'"）、适用指标/维度（上下文感知）
- 标注时上下文感知推荐规则（根据当前图表 Metric 如 "sales" 和 Dimension 如 "region" 自动筛选）
- 自动校验反馈：标注数据不符合规则时，前端 Warning（e.g., "数据 5% 不符合 >10% 规则"）
- 与标注系统联动：标注保存时注入规则到语料
- 与权限整合：不同角色对规则的访问控制（e.g., 业务角色只读）

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化）
- openspec/specs/backend-core/spec.md（多租户 tenant_id）
- openspec/specs/security-permissions/spec.md（权限过滤）
- openspec/specs/knowledge-corpus/spec.md（规则注入到语料）

## 交付物
1. **custom/rules/models.py**：SQLAlchemy 模型（BusinessRule + 关联表）
2. **custom/rules/api.py**：Flask Blueprint（CRUD 接口 + recommend + validate）
3. **custom/rules/recommend.py**：推荐逻辑函数（基于图表 Metric/Dimension）
4. **custom/rules/validate.py**：校验函数（表达式求值 + Warning 生成）
5. **frontend/src/rules/index.tsx**：React 页面（规则列表、CRUD 表单、推荐下拉）

## 关键实现点
### 1. 规则模型
- MySQL 表（business_rules）：
  ```sql
  CREATE TABLE business_rules (
      id INT AUTO_INCREMENT PRIMARY KEY,
      tenant_id VARCHAR(64) NOT NULL,
      name VARCHAR(100) NOT NULL,
      description TEXT,
      rule_type ENUM('threshold', 'pattern', 'classification', 'other'),
      expression TEXT,
      target TEXT,  -- e.g., "标注销售潜力"
      applicable_metrics JSON,  -- ["sales", "revenue"]
      applicable_dimensions JSON,  -- ["region", "time"]
      created_by INT,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
      updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  );
2. 上下文推荐

recommend.py 示例：Pythondef recommend_rules(chart_id):
    chart = Chart.query.get(chart_id)
    metrics = chart.metrics  # e.g., ['sales']
    dimensions = chart.dimensions  # e.g., ['region']
    return BusinessRule.query.filter(
        BusinessRule.applicable_metrics.overlap(metrics),
        BusinessRule.applicable_dimensions.overlap(dimensions)
    ).all()

3. 自动校验

validate.py 示例：Pythondef validate_data_against_rule(data_point, expression):
    # 使用 safe_eval 或 sympy 求值
    try:
        result = eval(expression, {"__builtins__": {}}, data_point)  # 安全 eval
        return result, None
    except Exception as e:
        return False, f"校验失败: {str(e)}"

4. 前端推荐 + Warning

index.tsx 示例：tsxuseEffect(() => {
  fetch(`/api/v1/rules/recommend?chart_id=${chartId}`).then(setSuggestedRules);
}, [chartId]);

const handleValidate = (selectedRule) => {
  fetch('/api/v1/rules/validate', { body: JSON.stringify({data: annotationData, expr: selectedRule.expression}) })
    .then(res => { if (!res.ok) showWarning(res.warning); });
};

测试说明

创建规则 → 在标注界面推荐 → 校验不符数据 → 注入到语料
权限测试：业务角色只能读规则

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 business-rules 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"
