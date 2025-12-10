# 工时结算与账单规范

- 模块：工时采集、单次工单结算、月度账单与报表导出
- 细节：支持多币种、工时归因、调整记录与审批流
- 输出：按项目/租户导出 PDF/CSV 报表，提供实时账期监控仪表盘

# 模块：billing-system（工时结算系统）

## 目标
- 标注工时自动累计
- 月结账单生成（Celery beat）
- 客户/内部报表

## 交付物
- custom/billing/ 目录
- annotation_tasks → annotation_bills
- 月结脚本

# 模块：billing-system（工时结算系统）

## 目标
- 支持业务/技术标注的工时自动累计（按小时/条计费 + 难度系数）
- 生成工时单（类似“结束行程”后审核/驳回）
- 月度自动结算账单（Celery beat 批处理，每月1号执行）
- 客户可见对账报表（本月标注消耗明细 + 导出 Excel）
- 内部人员奖励报表（工时排行榜 + 转为奖励金/积分）
- 与标注系统联动：标注结束时更新工时记录
- 灵活定价配置（annotation_types 表 + 个人费率覆盖）

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化）
- openspec/specs/backend-core/spec.md（多租户 tenant_id）
- openspec/specs/security-permissions/spec.md（权限过滤）
- openspec/specs/annotation-system/spec.md（工时记录联动）

## 交付物
1. **custom/billing/models.py**：SQLAlchemy 模型（AnnotationType + AnnotationBill + UserRate）
2. **custom/billing/api.py**：Flask Blueprint（start/finish 工时 + 报表接口）
3. **custom/billing/calculate.py**：费用计算函数（hourly/per_item + 系数）
4. **scripts/generate_monthly_bill.py**：月结批处理脚本（Celery beat）
5. **frontend/src/billing/index.tsx**：React 页面（客户报表、内部排行、账单下载）

## 关键实现点
### 1. 计费模型
- MySQL 表（annotation_types + annotation_bills + annotation_user_rates）：
  ```sql
  CREATE TABLE annotation_types (
      id INT AUTO_INCREMENT PRIMARY KEY,
      tenant_id VARCHAR(64) NOT NULL,
      name VARCHAR(50) NOT NULL,  -- "业务标注" / "技术标注"
      code VARCHAR(20) UNIQUE,
      unit_price DECIMAL(10,2) DEFAULT 0,
      pricing_mode ENUM('hourly','per_item') DEFAULT 'hourly',
      difficulty_factor DECIMAL(3,2) DEFAULT 1.00
  );

  CREATE TABLE annotation_bills (
      id BIGINT AUTO_INCREMENT PRIMARY KEY,
      tenant_id VARCHAR(64) NOT NULL,
      bill_no VARCHAR(32) UNIQUE,
      period VARCHAR(7) NOT NULL,  -- '2025-12'
      total_hours DECIMAL(12,4),
      total_items INT,
      total_amount DECIMAL(12,2),
      status ENUM('draft','confirmed','paid','cancelled') DEFAULT 'draft',
      paid_at DATETIME,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP
  );

  CREATE TABLE annotation_user_rates (
      user_id INT,
      tenant_id VARCHAR(64),
      annotation_type_id INT,
      unit_price DECIMAL(10,2),
      PRIMARY KEY (user_id, tenant_id, annotation_type_id)
  );
2. 费用计算

calculate.py 示例：Pythondef calculate_amount(task):
    atype = AnnotationType.query.get(task.annotation_type_id)
    if atype.pricing_mode == 'hourly':
        hours = task.duration_seconds / 3600
        return hours * atype.unit_price * atype.difficulty_factor
    else:
        return task.item_count * atype.unit_price

3. 月结批处理

generate_monthly_bill.py 示例：Python@celery.task
def generate_bill(tenant_id, year_month):
    tasks = AnnotationTask.query.filter_by(tenant_id=tenant_id, status='finished').filter(func.date_format(end_time, '%Y-%m') == year_month).all()
    total_amount = sum(calculate_amount(t) for t in tasks)
    bill = AnnotationBill(tenant_id=tenant_id, period=year_month, total_amount=total_amount)
    db.session.add(bill)
    db.session.commit()
Celery beat 配置：在 superset_config.py 中调度每月1号

4. 前端报表

index.tsx 示例：tsxconst BillingPage = () => {
  const [bills, setBills] = useState([]);
  useEffect(() => { fetch('/api/v1/billing/reports').then(setBills); }, []);
  
  return <Table dataSource={bills} columns={[{title: 'Period', dataIndex: 'period'}, {title: 'Amount', dataIndex: 'total_amount'}]} />;
};

测试说明

开始标注 → 结束 → 检查工时单生成
月末模拟运行批处理 → 查看账单
报表测试：导出 Excel + 权限控制（客户只见自己的）

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 billing-system 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"
