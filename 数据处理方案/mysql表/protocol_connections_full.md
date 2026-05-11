
## 协议连接表输出

输出侦测出的 PMU 协议 IEC104、IEC101 以及周期性 ICMP Ping 协议的连接信息，并支持周期性全量输出。

### 输出范围

- IEC104 连接
- IEC101 连接
- ICMP Ping 周期性探测连接（≥10 次）

### 连接信息字段（建议）

- 连接标识：`conn_id`
- 协议类型：`protocol`（IEC104/IEC101/PMU/ICMP_PING）
- 源/目的地址与端口：`src_ip`、`src_port`、`dst_ip`、`dst_port`
- 方向：`direction`（inbound/outbound）
- 发现时间：`first_seen`
- 最近活跃：`last_seen`
- 连接状态：`status`（active/inactive）
- 采样周期：`interval`（仅 ICMP Ping）
- 探测结果统计：`success_count`、`fail_count`（仅 ICMP Ping）

### 输出策略

- 周期性全量输出连接表。
- 输出周期：每 5 分钟。
- 每次输出覆盖当前所有有效连接记录。

### 全量表差异比较

- 每次输出带 `snapshot_id` 与 `snapshot_time`，用于标识批次。
- 以 `conn_id` 为主键做差异比对：
  - 新增：新批次存在、旧批次不存在。
  - 删除：旧批次存在、新批次不存在。
  - 变更：两批次 `conn_id` 相同但关键字段变化。
- 建议增加 `row_hash`（对关键字段做哈希）以快速判断变更。
- 差异结果可写入对账表：`protocol_connections_diff`（含 `change_type`、`change_time`、`snapshot_id`）。

### 输出形式（示例）

- 写入 MySQL 表。
- 表名建议：`protocol_connections_full`


