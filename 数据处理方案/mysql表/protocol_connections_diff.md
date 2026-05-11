# 协议连接差异表（待议）

基于协议连接全量表，输出差异表，用于标识每个快照批次的新增、删除与变更。

- 当前 MySQL 表名：`protocol_connections_diff`

## 全量表差异比较

- 每次输出带 `snapshot_id` 与 `snapshot_time`，用于标识批次。
- 以 `conn_id` 为主键做差异比对：
  - 新增：新批次存在、旧批次不存在。
  - 删除：旧批次存在、新批次不存在。
  - 变更：两批次 `conn_id` 相同但关键字段变化。
- 建议增加 `row_hash`（对关键字段做哈希）以快速判断变更。
- 差异结果写入对账表：`protocol_connections_diff`（含 `change_type`、`change_time`、`snapshot_id`）。

## 差异表字段（建议）

- `diff_id`
- `snapshot_id`
- `snapshot_time`
- `conn_id`
- `change_type`（new/update/delete）
- `change_time`
- `row_hash_old`
- `row_hash_new`

