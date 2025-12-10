# 知识语料管理规范

- 目标：建立语料治理闭环，包含治理前质检、治理后归档与 BI 反哺
- 关键流程：语料接入 → 质量打分 → 补充标签 → 入库/归档 → 数据集共享
- 输出：可视化治理报告、BI 看板指标、反哺业务规则的语料洞察

# 模块：knowledge-corpus（知识语料管理）

## 目标
- 治理前/后语料分类存储
- 反哺 Superset BI 分析（新数据集）
- 语料对比展示
- MongoDB 存储实际内容

## 交付物
- custom/knowledge/ 目录
- knowledge_corpus 表 CRUD
- MongoDB 操作封装

# 模块：knowledge-corpus（知识语料管理）

## 目标
- 支持治理前语料（原始导入数据）和治理后语料（标注 + 规则注入后）的分类存储和管理
- 语料分类（按业务领域分桶，使用 category_id）
- 治理后语料在图表中清晰展示规则注入效果（e.g., hover tooltip 显示“规则注入: 阈值校验通过”）
- 反哺 Superset BI 分析：治理后语料作为新数据源（e.g., 创建虚拟数据集分析“规则触发频次”或“业务线异常数据分布”）
- MongoDB 存储实际语料内容（结构化 JSON），MySQL 存元信息
- 支持语料对比展示（治理前 vs 后）

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化）
- openspec/specs/backend-core/spec.md（多租户 tenant_id）
- openspec/specs/security-permissions/spec.md（权限过滤）

## 交付物
1. **custom/knowledge/models.py**：SQLAlchemy 模型（KnowledgeCorpus + 关联表）
2. **custom/knowledge/api.py**：Flask Blueprint（CRUD 接口 + 对比 API）
3. **custom/knowledge/mongo_ops.py**：MongoDB 操作封装（insert/update/query/delete）
4. **custom/knowledge/bi_integration.py**：反哺 BI 函数（创建 Superset 虚拟数据集）
5. **frontend/src/knowledge/index.tsx**：React 页面（语料列表、对比视图、注入展示）

## 关键实现点
### 1. 语料存储
- MySQL 元表（knowledge_corpus）：
  ```sql
  CREATE TABLE knowledge_corpus (
      id BIGINT AUTO_INCREMENT PRIMARY KEY,
      tenant_id VARCHAR(64) NOT NULL,
      name VARCHAR(100) NOT NULL,
      type ENUM('pre_governance', 'post_governance') NOT NULL,
      category_id INT,
      mongo_id VARCHAR(64),
      rule_injected JSON,
      version INT DEFAULT 1,
      snapshot_rules JSON,
      description TEXT,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
      updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  );

MongoDB 实际内容示例：Pythondef save_to_mongo(corpus_data):
    return mongo_db.corpus.insert_one({'tenant_id': g.tenant_id, 'data': corpus_data}).inserted_id

2. 治理前后对比

API：/api/v1/knowledge/compare?pre_id=1&post_id=2
返回 Diff JSON（使用 difflib 或类似库）

3. 规则注入展示

在 Superset 图表插件中添加 tooltip：JavaScripttooltip: {
    formatter: function() {
        return this.point.rule_injected ? `规则注入: ${this.point.rule_injected}` : '';
    }
}

4. 反哺 BI

bi_integration.py 示例：Pythondef create_virtual_dataset(corpus_id):
    corpus = KnowledgeCorpus.query.get(corpus_id)
    # 使用 Superset API 创建虚拟 SQL 数据集
    superset_api.create_dataset(f"SELECT * FROM virtual_corpus WHERE mongo_id='{corpus.mongo_id}'")

测试说明

导入原始语料 → 标注生成治理后 → 查询对比 API
权限测试：业务角色只能见治理后语料
BI 测试：创建仪表盘分析治理后语料

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 knowledge-corpus 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"