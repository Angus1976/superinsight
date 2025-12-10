# 混合部署规范

- 场景：容器 + 本地机房双活，需同步监控与缓存状态
- 要素：服务发现、连接状态路由、缓存一致性策略、指标采集
- 约束：边缘节点需提供低带宽运行模式与故障自愈机制

# 模块：hybrid-deployment（混合部署）

## 目标
- 连接状态监控面板
- Redis 热数据缓存
- Frp/ZeroTier 封装

## 交付物
- custom/hybrid/ 目录
- 监控 API + 报警

# 模块：hybrid-deployment（混合部署）

## 目标
- 支持混合部署模式：应用层在腾讯云托管，数据源/语料留在客户内网（永不落云端）
- 内网穿透方案：Frp / ZeroTier / 专线 / 轻量 Connector（Go 编写，仅转发 SQL）
- 连接状态监控面板（SaaS 后台显示“本地数据源连接状态” + 延迟指标）
- Redis 热数据缓存策略（针对高延迟风险，定时同步脱敏热数据到云端缓存）
- 报警机制：连接中断时推送警报（e.g., 邮件/企业微信）

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化）
- openspec/specs/backend-core/spec.md（多租户 tenant_id + Docker）
- openspec/specs/security-permissions/spec.md（脱敏 + 权限）

## 交付物
1. **custom/hybrid/config.py**：混合部署配置（环境变量 + 穿透工具设置）
2. **custom/hybrid/monitor.py**：连接状态检查函数 + 报警逻辑
3. **custom/hybrid/cache.py**：Redis 热缓存策略（同步/查询 函数）
4. **custom/hybrid/connector.go**：轻量 Connector 示例（Go 源码，非必须编译）
5. **frontend/src/hybrid/index.tsx**：React 页面（连接状态监控面板 + 警报列表）

## 关键实现点
### 1. 内网穿透
- config.py 示例（Frp 配置）：
  ```python
  if os.getenv('DEPLOY_MODE') == 'hybrid':
      frp_client = FrpClient(server='frp.server.com:7000', token='xxx')
      frp_client.tunnel(local_port=3306, remote_port=3306)  # MySQL 穿透
2. 连接状态监控

monitor.py 示例：Pythondef check_connection():
    try:
        ping = test_ping('customer.db.internal')  # 或数据库连接测试
        return {'connected': True, 'latency': ping.latency}
    except:
        send_alert('连接中断')
        return {'connected': False, 'error': 'Timeout'}
API：/api/v1/hybrid/status 返回 JSON

3. Redis 热缓存

cache.py 示例：Pythondef sync_hot_data():
    hot_queries = get_frequent_queries()  # 从日志获取
    for q in hot_queries:
        data = execute_query(q, desensitize=True)  # 脱敏
        redis.set(f'hot:{q.hash}', json.dumps(data), ex=3600)  # 1小时过期
Celery 定时任务：每5min 同步

4. 前端面板

index.tsx 示例：tsxconst HybridMonitor = () => {
  const [status, setStatus] = useState({});
  useInterval(() => { fetch('/api/v1/hybrid/status').then(setStatus); }, 60000);
  
  return <Card title="连接状态">
    <p>连接: {status.connected ? '正常' : '中断'}</p>
    <p>延迟: {status.latency}ms</p>
  </Card>;
};

测试说明

配置混合模式 → 检查连接状态 API
模拟中断 → 验证警报发送
查询热数据 → 检查 Redis 缓存命中

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 hybrid-deployment 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"