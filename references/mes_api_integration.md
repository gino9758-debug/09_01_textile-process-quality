# 飞书多维表格数据接入规范

## 1. 接入架构概述

针织服装质量实时监测数据来源于飞书多维表格（Base）：

- **飞书多维表格（Base）**：验货报告、仓检报告、包材检验记录、质量分析数据
- **飞书任务**：质量异常整改任务的跟踪与闭环

**多维表格列表（有棵树采购品质知识库 Base ID: Uu3SbsoeXa5mziscDuXcRUMmn6e）**

| 表名 | table_id | 用途 |
|------|----------|------|
| 认证QC-自营-厂检-验货报告 | tblIg1zQbxZHtbcz | 自营款厂检验货记录 |
| 认证QC-联营-厂检-查货报告 | tblaTl84D3BsliEc | 联营款厂检记录 |
| QI-自营-仓检-验货报告 | tbl4MOTwxWelentS | 仓检验货记录 |
| QC-自营-厂检-验货报告 | tblLgEsJ4SHMBKPz | QC自营厂检验货 |
| QC-联营-厂检-验货报告 | tblngaTU2bV0eWqt | QC联营厂检验货 |
| 包材-认证QC-厂检-查货报告 | tbll0XnyqcqT6Jjy | 包材厂检记录 |
| 包材-QC-包材厂-厂检-查货报告 | tblewGwO8YM946lL | 包材QC报告 |
| 异常闭环追踪 | tblnU5284kpedfWv | 质量异常整改进度 |
| 管理-供应商质量改善台账 | tblb1bIeJQTnOLzv | 供应商质量改善记录 |
| QA-批样抽检台账 | tblx2unkwDLNK0M9 | 批样抽检记录 |

> **注意**：原 URL 中的表 ID（`wkfU2WE1BLs4C3Rl`）已不在当前 Base 中，如需查找特定数据表，请使用上表中的正确 table_id，或通过 `lark-cli base +table-list --base-token Uu3SbsoeXa5mziscDuXcRUMmn6e` 列出所有可用表。

## 2. 接入方式选择

### 2.1 飞书 Base API（推荐）

- **适用场景**：通过飞书开放平台 API 读取多维表格记录
- **协议**：HTTPS/JSON
- **认证方式**：飞书应用身份（TAT），使用 `lark-cli` 工具调用
- **典型接口**：
  - `lark-cli base +record-list` — 拉取表格记录（支持按字段筛选）
  - `lark-cli base +record-search` — 按关键词搜索记录
  - `lark-cli base +field-list` — 获取表格字段列表

### 2.2 lark-cli 调用规范

```bash
# 列出所有可用表格
lark-cli base +table-list --as bot --base-token Uu3SbsoeXa5mziscDuXcRUMmn6e

# 获取表格字段列表
lark-cli base +field-list --as bot --base-token Uu3SbsoeXa5mziscDuXcRUMmn6e --table-id <TABLE_ID>

# 按筛选条件拉取记录（示例：验货不合格记录）
lark-cli base +record-list --as bot --base-token Uu3SbsoeXa5mziscDuXcRUMmn6e --table-id <TABLE_ID> --filter '{"conjunction":"and","conditions":[{"field_name":"验货结论","operator":"is_not","value":["合格"]}]}' --sort "[{"field_name":"最后更新时间","desc":true}]" --limit 100
```

## 3. 数据字段映射规范

### 3.1 验货报告表统一字段名

| 统一字段名 | 中文含义 | 典型源字段名 | 数据类型 |
|-----------|---------|------------|---------|
| record_time | 记录时间 | 最后更新时间/验货日期 | datetime |
| style_no | 款号 | 款号/款名 | string |
| factory_name | 工厂名称 | 工厂/供应商 | string |
| batch_no | 批次号 | 批次/订单号 | string |
| business_type | 业务类型 | 自营/联营 | string |
| qc_result | 验货结论 | 合格/不合格/整改 | string |
| defect_type | 缺陷类型 | 不合格项/缺陷描述 | string |
| defect_count | 不合格数量 | 不合格数量 | int |
| sample_size | 抽检数量 | 抽检数量 | int |
| defect_rate | 不合格率 | 不合格率/% | float |
| inspector | 验货员 | QC/验货员 | string |
| remark | 备注 | 备注 | string |

### 3.2 仓检报告表统一字段名

| 统一字段名 | 中文含义 | 典型源字段名 | 数据类型 |
|-----------|---------|------------|---------|
| record_time | 到货日期 | 到货日期/最后更新时间 | datetime |
| style_no | 款号 | 款号/款名 | string |
| factory_name | 工厂名称 | 工厂/供应商 | string |
| qi_result | 仓检结论 | 仓检结果/合格判定 | string |
| defect_desc | 不合格描述 | 不合格描述/备注 | string |
| defect_quantity | 不合格数量 | 不合格数量 | int |
| total_quantity | 到货总量 | 到货总量 | int |
| qi_defect_rate | 仓检不合格率 | 不合格率/% | float |
| warehouse | 仓库 | 仓库/仓库名称 | string |

### 3.3 包材检验表统一字段名

| 统一字段名 | 中文含义 | 典型源字段名 | 数据类型 |
|-----------|---------|------------|---------|
| record_time | 检验日期 | 检验日期/最后更新时间 | datetime |
| packaging_type | 包材类型 | 包材类型/品名 | string |
| factory_name | 供应商 | 供应商/包材厂 | string |
| batch_no | 批次号 | 批次/订单号 | string |
| inspection_result | 检验结论 | 合格/不合格 | string |
| error_type | 错误类型 | 标签错误/款号错误/尺寸偏差 | string |
| error_description | 错误描述 | 不良描述/备注 | string |
| error_quantity | 不良数量 | 不良数量 | int |

## 4. 数据拉取策略

### 4.1 准实时批量模式（推荐）

- **频率**：5~15分钟/次
- **方式**：按时间窗口批量拉取，使用 `last_modified_time` 过滤
- **参数**：`start_time = last_fetch_time`，`end_time = now`，`limit = 500`
- **去重策略**：以 `record_id` 为唯一键，本地缓存已处理记录

### 4.2 按条件筛选模式

```bash
# 示例：拉取近30天内所有验货不合格记录
lark-cli base +record-list \
  --as bot \
  --base-token Uu3SbsoeXa5mziscDuXcRUMmn6e \
  --table-id tblIg1zQbxZHtbcz \
  --filter '{"conjunction":"and","conditions":[{"field_name":"验货结论","operator":"is_not","value":["合格"]}]}' \
  --sort "[{"field_name":"最后更新时间","desc":true}]" \
  --limit 100
```

### 4.3 历史回溯模式

- **触发条件**：异常发生后追溯分析，或用户指定款号/工厂/时间段
- **时间范围**：通常回溯近3~6个月数据
- **粒度**：按批次/款号聚合，不做单件级降采样

## 5. 数据质量校验

每次拉取数据后执行以下校验：

| 校验项 | 规则 | 异常处理 |
|-------|------|---------|
| 时间戳合理性 | 记录时间不得早于系统上线时间，不得晚于当前时间 | 标记异常时间记录 |
| 数值合理性 | 不合格率在 0%~100% 范围内 | 标记越界值 |
| 字段完整性 | 关键字段（款号/工厂/结论）不得为空 | 空值率>5%时告警 |
| 款号格式 | 款号格式应符合公司规范 | 标记格式异常 |
| 记录一致性 | 同批次在验货表和仓检表中的款号应一致 | 标记不一致记录 |

## 6. 接入配置模板

```yaml
connection:
  type: FEISHU_BASE
  base_token: "Uu3SbsoeXa5mziscDuXcRUMmn6e"
  auth_type: TAT  # 应用身份认证
  refresh_interval_sec: 600  # 10分钟/次

tables:
  qc_selfoperated: "tblIg1zQbxZHtbcz"  # 认证QC-自营-厂检-验货报告
  qc_franchise: "tblaTl84D3BsliEc"      # 认证QC-联营-厂检-查货报告
  qi_warehouse: "tbl4MOTwxWelentS"       # QI-自营-仓检-验货报告
  packaging_inspection: "tbll0XnyqcqT6Jjy" # 包材-认证QC-厂检-查货报告
  defect_tracking: "tblnU5284kpedfWv"     # 异常闭环追踪
  supplier_quality: "tblb1bIeJQTnOLzv"     # 管理-供应商质量改善台账

alert:
  defect_rate_threshold_pct: 5    # 不合格率告警阈值（%）
  null_rate_threshold_pct: 5     # 空值率告警阈值（%）
  consecutive_defects_count: 3   # 连续不合格批次告警阈值
```

## 7. 安全与权限

- 使用飞书应用身份（TAT）读取数据，无需用户个人授权
- 仅使用只读权限，禁止任何写入/修改操作（写入通过飞书任务系统）
- 所有数据读取操作需记录审计日志（时间、表名、拉取记录数）
