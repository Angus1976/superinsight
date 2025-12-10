# 语料版本管理规范

- 要求：提供语料快照、差异对比、回滚能力，覆盖所有结构化与非结构化内容
- 接口：支持按标签/时间查询版本、批量恢复、变更审核记录
- 安全性：快照仅限授权人员下载，通过加密通道同步

# 模块：version-control（版本管理）

## 目标
- 语料快照（规则版本保存）
- 批量回滚
- 回滚日志

## 交付物
- custom/version/ 目录
- 快照/恢复函数
- 回滚 API

# 模块：version-control（版本管理）

## 目标
- 支持知识语料的版本快照（保存当时注入的规则版本，避免规则修改影响历史语料）
- 批量回滚机制：如果批量注入规则失误，一键回滚到指定版本
- 回滚日志记录（who/when/why）
- 与语料管理联动：生成治理后语料时自动创建快照
- 与权限整合：仅技术/管理员角色可回滚

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化）
- openspec/specs/backend-core/spec.md（多租户 tenant_id）
- openspec/specs/security-permissions/spec.md（权限过滤 + 角色矩阵）
- openspec/specs/knowledge-corpus/spec.md（语料模型 + MongoDB）
- openspec/specs/annotation-system/spec.md（标注时创建快照）

## 交付物
1. **custom/version/models.py**：SQLAlchemy 模型（CorpusRollbackLog）
2. **custom/version/api.py**：Flask Blueprint（snapshot/create + rollback 接口）
3. **custom/version/snapshot.py**：快照生成/恢复函数（MongoDB 备份/还原）
4. **custom/version/rollback.py**：回滚逻辑（批量处理 + 日志）
5. **frontend/src/version/index.tsx**：React 页面（版本列表、回滚按钮、日志查看）

## 关键实现点
### 1. 版本模型
- MySQL 表（corpus_rollback_logs）：
  ```sql
  CREATE TABLE corpus_rollback_logs (
      id BIGINT AUTO_INCREMENT PRIMARY KEY,
      tenant_id VARCHAR(64) NOT NULL,
      corpus_id BIGINT NOT NULL,
      version_from INT,
      version_to INT,
      rolled_back_at DATETIME DEFAULT CURRENT_TIMESTAMP,
      user_id INT,
      reason TEXT
  );

knowledge_corpus 表已有 version + snapshot_rules 字段

2. 快照生成

snapshot.py 示例：Pythondef create_snapshot(corpus_id):
    corpus = KnowledgeCorpus.query.get(corpus_id)
    snapshot = mongo_db.corpus.find_one({'_id': corpus.mongo_id}).copy()
    snapshot['version'] = corpus.version
    mongo_db.snapshots.insert_one(snapshot)  # 单独集合存储快照
    return snapshot['_id']

3. 批量回滚

rollback.py 示例：Pythondef rollback_corpus(corpus_id, target_version, reason):
    corpus = KnowledgeCorpus.query.get(corpus_id)
    snapshot = mongo_db.snapshots.find_one({'corpus_id': corpus_id, 'version': target_version})
    if snapshot:
        mongo_db.corpus.update_one({'_id': corpus.mongo_id}, {'$set': snapshot['data']})
        corpus.version = target_version
        log = CorpusRollbackLog(tenant_id=g.tenant_id, corpus_id=corpus_id, version_from=corpus.version, version_to=target_version, user_id=g.user.id, reason=reason)
        db.session.add(log)
        db.session.commit()
        return True
    return False, "Snapshot not found"

4. 前端界面

index.tsx 示例：tsxconst VersionPage = ({ corpusId }) => {
  const [versions, setVersions] = useState([]);
  useEffect(() => { fetch(`/api/v1/version/list?corpus_id=${corpusId}`).then(setVersions); }, [corpusId]);
  
  const handleRollback = (version) => {
    fetch('/api/v1/version/rollback', { method: 'POST', body: JSON.stringify({corpus_id: corpusId, version_to: version, reason: 'Fix error'}) });
  };
  
  return <List dataSource={versions} renderItem={v => <Item>Version {v.version} <Button onClick={() => handleRollback(v.version)}>Rollback</Button></Item>} />;
};

测试说明

生成语料 → 修改规则 → 回滚到旧版本 → 检查语料恢复
日志测试：查看回滚记录
权限测试：业务角色无回滚按钮

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 version-control 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"