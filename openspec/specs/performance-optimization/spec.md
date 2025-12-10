# 性能优化规范

- 聚焦：分片、归档、数据瓦片化，以降低查询与存储成本
- 手段：智能路由、异步归档策略、查询缓存与指标限幅器
- 监控：设置热点追踪、慢查询分析与容量预警

# 模块：performance-optimization（性能优化）

## 目标
- MongoDB 分片（tenant_id）
- 历史语料归档
- 大宽表标注抽样/瓦片化

## 交付物
- custom/performance/ 目录
- archive_old_corpus.py
- 分片初始化脚本

# 模块：performance-optimization（性能优化）

## 目标
- 应对大规模语料增长：MongoDB 分片（以 tenant_id 为分片键）+ 历史语料冷热分离/归档
- 大宽表（百万级数据点）标注性能优化：前端抽样 + 瓦片化（tile-based）渲染，避免卡顿
- 历史标注/语料归档：超过一定天数的冷数据移到归档存储（OSS/S3），保留元信息
- 整体系统性能监控：关键接口响应时间、MongoDB 查询耗时、Redis 命中率

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化）
- openspec/specs/backend-core/spec.md（Docker + Celery）
- openspec/specs/knowledge-corpus/spec.md（MongoDB 语料存储）
- openspec/specs/annotation-system/spec.md（大宽表标注画布）

## 交付物
1. **custom/performance/sharding.py**：MongoDB 分片初始化脚本（在 init_tenant.py 中调用）
2. **custom/performance/archive.py**：语料归档函数 + Celery 定时任务
3. **custom/performance/tile.py**：大宽表瓦片化逻辑（后端生成瓦片，前端请求）
4. **custom/performance/monitor.py**：性能监控指标收集（Prometheus 或自定义）
5. **scripts/archive_old_corpus.py**：归档脚本（可独立运行）
6. **frontend/src/components/AnnotationTool/tile.tsx**：瓦片化标注组件（Konva + 抽样）

## 关键实现点
### 1. MongoDB 分片
- sharding.py 示例：
  ```python
  def init_sharding():
      mongo_db.admin.command('enableSharding', 'superinsight')
      mongo_db.admin.command('shardCollection', 'superinsight.corpus', key={'tenant_id': 'hashed'})
      mongo_db.admin.command('shardCollection', 'superinsight.snapshots', key={'tenant_id': 'hashed'})
2. 历史归档

archive.py 示例：Python@celery.task
def archive_old_corpus(tenant_id, days=365):
    cutoff = datetime.now() - timedelta(days=days)
    old_corpus = KnowledgeCorpus.query.filter(
        KnowledgeCorpus.tenant_id == tenant_id,
        KnowledgeCorpus.updated_at < cutoff
    ).all()
    
    for c in old_corpus:
        mongo_db.archive.insert_one(mongo_db.corpus.find_one({'_id': c.mongo_id}))
        mongo_db.corpus.delete_one({'_id': c.mongo_id})
        c.status = 'archived'
    db.session.commit()

3. 大宽表瓦片化

tile.py 示例：Pythondef get_tile_data(chart_id, zoom, x, y):
    # 根据 zoom/x/y 计算可见区域
    data = execute_query(f"SELECT * FROM dataset WHERE chart_id={chart_id} LIMIT 10000")
    return data  # 返回抽样或聚合后的点
前端 tile.tsx：tsxconst TileAnnotation = ({ chartId }) => {
  // 使用 Konva + 动态加载瓦片
  return <Layer>{/* 渲染瓦片 */}</Layer>;
};

4. 性能监控

monitor.py 示例：Pythondef record_metrics():
    metrics = {
        'mongo_query_time': mongo_db.command('profile')[-1]['millis'],
        'redis_hit_rate': redis.info('stats')['keyspace_hits'] / (redis.info('stats')['keyspace_hits'] + redis.info('stats')['keyspace_misses'])
    }
    # 可推送到 Prometheus 或日志

测试说明

导入百万级语料 → 验证分片生效（mongo shell 查看 shards）
模拟老数据 → 运行归档任务 → 检查数据移到 archive 集合
在大宽表上标注 → 验证前端不卡顿（瓦片加载）

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 performance-optimization 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"