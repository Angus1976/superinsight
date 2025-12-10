# 协作标注系统规范

- 核心交互：任务分配、实时协作视图、质量复核与变更记录
- 工时计费：自动关联标注工时、支持校准因子、导出月度账单明细
- 依赖：与权限体系联动，仅开放给经过认证的操作员

# 模块：annotation-system（协作标注系统）

## 目标
- 图表上文字/箭头/矩形/高亮标注
- 实时协同（WebSocket）
- 业务/技术标注区分
- 标注保存时自动生成治理后语料
- 规则校验 Warning
- 工时自动计时

## 交付物
- custom/annotations/ 完整目录
- Flask Blueprint + SocketIO
- 前端 AnnotationTool（Konva.js）

## 关键实现点
- 标注保存 → 调用 business-rules 校验 → 生成 knowledge-corpus 条目
- 工时记录写入 annotation_tasks 表

# 模块：annotation-system（协作标注系统）

## 目标
- 支持图表上的协作标注（文字、箭头、矩形、高亮、实时多人协同 via WebSocket）
- 业务标注 vs 技术标注区分（不同费率 + 难度系数）
- 标注过程自动计时（精确到秒，暂停/结束按钮）
- 与知识语料联动：标注保存时生成治理后语料（带规则注入 + 快照）
- 与规则库联动：标注时推荐规则 + 自动校验 Warning
- 与工时系统联动：标注操作记录到 annotation_tasks 表，用于结算
- 前端使用 Konva.js 实现画布交互，支持大宽表抽样（后续性能模块优化）

## 依赖
- openspec/project.md（全局总览）
- openspec/changes/project-init/plan.md（初始化）
- openspec/specs/backend-core/spec.md（多租户 tenant_id）
- openspec/specs/security-permissions/spec.md（权限过滤 + 角色矩阵）
- openspec/specs/knowledge-corpus/spec.md（生成治理后语料）
- openspec/specs/business-rules/spec.md（规则推荐 + 校验）

## 交付物
1. **custom/annotations/models.py**：SQLAlchemy 模型（ChartAnnotation + AnnotationTask）
2. **custom/annotations/api.py**：Flask Blueprint（start/finish/save 接口）
3. **custom/annotations/websocket.py**：SocketIO 实时协同逻辑
4. **custom/annotations/linkage.py**：标注联动函数（生成语料 + 规则注入 + 校验）
5. **frontend/src/components/AnnotationTool/index.tsx**：React 组件（画布、类型选择、计时器、规则下拉、Warning 弹窗）

## 关键实现点
### 1. 标注模型
- MySQL 表（chart_annotations + annotation_tasks）：
  ```sql
  CREATE TABLE chart_annotations (
      id BIGINT AUTO_INCREMENT PRIMARY KEY,
      tenant_id VARCHAR(64) NOT NULL,
      chart_id INT,
      dashboard_id INT,
      user_id INT,
      mongo_id VARCHAR(64),
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP
  );

  CREATE TABLE annotation_tasks (
      id BIGINT AUTO_INCREMENT PRIMARY KEY,
      tenant_id VARCHAR(64) NOT NULL,
      annotation_id BIGINT NOT NULL,
      user_id INT NOT NULL,
      annotation_type_id INT NOT NULL,
      chart_id INT,
      dashboard_id INT,
      start_time DATETIME(3) NOT NULL,
      end_time DATETIME(3),
      duration_seconds INT,
      item_count INT DEFAULT 1,
      status ENUM('running','paused','finished','rejected') DEFAULT 'running',
      remark TEXT,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
      updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  );
2. 实时协同

websocket.py 示例：Python@socketio.on('annotate')
def handle_annotate(data):
    room = data['chart_id']
    emit('update_annotation', data, room=room)

3. 联动生成语料

linkage.py 示例：Pythondef save_annotation(data):
    # 规则校验
    valid, warning = validate_data_against_rule(data['data_point'], data['rule_expr'])
    if not valid:
        return {'warning': warning}, 400
    
    # 生成语料
    corpus = KnowledgeCorpus(type='post_governance', mongo_id=save_to_mongo(data))
    db.session.add(corpus)
    
    # 注入规则快照
    corpus.snapshot_rules = get_rule_snapshot(data['rule_id'])
    
    # 记录工时
    task = AnnotationTask(...)  # 更新 duration
    db.session.add(task)
    db.session.commit()
    
    return {'corpus_id': corpus.id}

4. 前端交互

index.tsx 示例：tsxconst AnnotationTool = () => {
  const [timer, setTimer] = useState(0);
  useEffect(() => { /* 计时逻辑 */ }, []);
  
  const handleSave = () => {
    // 调用 API 保存 + 校验
  };
  
  return <Canvas onAnnotate={handleAnnotate} />;
};

测试说明

进入图表 → 开始标注（选类型） → 画标注 → 保存（见 Warning 或成功生成语料）
多用户测试：WebSocket 实时更新
工时测试：结束时检查 duration_seconds

AI 生成 Prompt 模板
复制本 spec.md 给 AI：
"根据以下 spec.md 内容，生成 annotation-system 模块所有交付物代码。输出每个文件路径 + 完整代码。完成后输出 '模块完成'。"