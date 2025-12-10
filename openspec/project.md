# 项目总览

## 目的
建立一套支持多租户、中文优先、可视化治理的知识语料全生命周期平台，涵盖管理员后台、协作标注、语料治理、业务规则、结算与混合部署等能力。

## 主要模块
- 管理员后台：租户/角色/权限、操作监控、Superset 仪表板集成
- 协作标注系统：任务分发、工时计费、审核机制
- 知识语料管理：质量治理、归档、BI 反哺
- 业务规则库：CRUD、智能推荐、命中校验
- 语料版本管理：快照、回滚、差异对比
- 工时结算：账单导出、多币种、审批链
- 混合部署：监控 + 缓存 + 连接状态保障
- 性能优化：分片、瓦片化、归档策略
- 安全权限：角色矩阵、脱敏、审计
- 后端核心：多租户、中文搜索、Superset 定制

## 里程碑
1. Phase 0：完成模块蓝图与 Agent 责任分配
2. Phase 1：实现管理员控制台 + 协作标注 + 语料治理
3. Phase 2：补全结算、规则、版本与混合部署能力
4. Phase 3：规模优化、监控、性能与安全强化

# SuperInsight 项目全局总览

## 项目名称
SuperInsight 企业级数据洞察平台（基于 Apache Superset 4.0+ 深度定制）

## 核心目标
- 支持 SaaS、多租户、私有化、混合部署
- 完全中文 + 业务化术语
- 业务人员可在图表上直接协作标注
- 标注过程自动生成治理后知识语料
- 业务规则库智能推荐与校验
- 工时自动计费 + 月结账单
- 语料版本快照 + 回滚
- 混合部署连接监控 + 热缓存
- 性能分片、归档、大宽表瓦片化

## 技术栈
- 后端：Python 3.11 + Flask + Superset 4.0.2+
- 数据库：MySQL 8.0 + MongoDB 7.x + Redis 7.x
- 前端：React + Konva.js（标注画布）
- 部署：Docker Compose + 腾讯云 TCB/云托管

## 模块清单（对应 specs/ 目录）
- backend-core：多租户、Superset 定制、中文
- security-permissions：权限体系 + 角色矩阵 + 脱敏
- knowledge-corpus：治理前/后语料管理 + 反哺 BI
- business-rules：规则库 CRUD + 智能推荐/校验
- annotation-system：图表标注 + 工时计费 + 语料联动
- billing-system：结算 + 月结 + 报表
- version-control：语料快照 + 回滚
- hybrid-deployment：混合部署监控 + 缓存
- performance-optimization：MongoDB 分片、归档、瓦片化
- admin-system：管理员后台（租户、连接状态等）

## 推荐开发顺序
1. project.md → backend-core → security-permissions
2. knowledge-corpus → business-rules → annotation-system
3. billing-system → version-control
4. hybrid-deployment → performance-optimization
5. admin-system（最后收尾）

# SuperInsight 项目全局总览

## 项目名称
SuperInsight 企业级数据洞察平台（基于 Apache Superset 4.0+ 深度定制）

## 核心目标
- 支持 SaaS、多租户、私有化、混合部署（数据永远留在客户侧）
- 完全中文界面 + 业务化术语（“切片”→“图表”，“数据库”→“数据源”等）
- 业务人员可在图表上直接协作标注（文字、箭头、矩形、高亮、实时协同）
- 标注过程自动生成治理后知识语料（标注即治理）
- 嵌入业务规则库，支持上下文感知推荐 + 自动校验 Warning
- 业务/技术标注双模式，自动计工时、按量结算、月度账单
- 知识语料版本管理（快照 + 批量回滚）
- 混合部署连接状态监控 + Redis 热缓存
- 性能优化（MongoDB 分片、历史归档、大宽表标注抽样/瓦片化）
- 细粒度权限体系 + 数据脱敏 + 角色矩阵

## 技术栈（2025年12月最新稳定版）
- 后端：Python 3.11 + Flask + Apache Superset 4.0.2+
- 数据库：
  - MySQL 8.0（元数据、权限、工时、规则、版本）
  - MongoDB 7.x（标注内容、治理前后语料）
  - Redis 7.x（缓存 + Celery + 混合部署热缓存）
- 前端：React + Konva.js（标注画布）+ Ant Design
- 部署：
  - Docker Compose（私有化）
  - 腾讯云 TCB / 云托管（SaaS）
  - 内网穿透：Frp / ZeroTier / 专线

## 模块清单（对应 specs/ 目录）
| 模块名                     | 目录路径                          | 主要职责                                                                 |
|----------------------------|-----------------------------------|--------------------------------------------------------------------------|
| backend-core               | specs/backend-core                | Superset 基础定制、多租户、强制中文、核心配置                             |
| security-permissions       | specs/security-permissions        | 四层权限 + 角色矩阵 + 语料权限 + 数据脱敏                               |
| knowledge-corpus           | specs/knowledge-corpus            | 治理前/后知识语料管理 + 反哺 BI 分析                                    |
| business-rules             | specs/business-rules              | 业务规则库 CRUD + 上下文推荐 + 自动校验                                  |
| annotation-system          | specs/annotation-system           | 图表协作标注 + 工时计时 + 语料联动生成 + 规则校验                        |
| billing-system             | specs/billing-system              | 工时结算 + 月结账单 + 客户/内部报表                                      |
| version-control            | specs/version-control             | 语料快照 + 批量回滚 + 回滚日志                                           |
| hybrid-deployment          | specs/hybrid-deployment           | 混合部署连接监控 + Redis 热缓存 + 报警                                   |
| performance-optimization   | specs/performance-optimization    | MongoDB 分片、历史归档、大宽表标注抽样/瓦片化                            |
| admin-system               | specs/admin-system                | 管理员后台（租户管理、连接状态总览、账单管理、系统监控）                 |

## 推荐开发顺序（建议严格按此顺序）
1. project.md（本文件）  
2. backend-core  
3. security-permissions  
4. knowledge-corpus  
5. business-rules  
6. annotation-system  
7. billing-system  
8. version-control  
9. hybrid-deployment  
10. performance-optimization  
11. admin-system（最后收尾）

## 交付标准（每个 spec.md 完成后）
- 完整代码文件列表（路径 + 文件名）
- 每个文件的完整代码（可直接复制）
- 运行/测试说明
- 依赖的其他模块说明

## 后续可扩展方向（2.0 计划）
- 企业微信/钉钉深度集成（审批、红包发放）
- AI 自动标注建议（基于规则库智能框选）
- 语料版本 Diff 视图
- 标注数据用于模型微调闭环
- WebGL 加速大宽表标注

本文件为项目唯一入口，所有开发人员和 AI Agent 均应以此为准。

