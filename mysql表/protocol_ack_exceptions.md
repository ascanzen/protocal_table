# IEC101/IEC104 业务数据发送确认状态异常记录

记录 IEC101/IEC104 业务数据发送与确认（ACK/否认/超时）的状态，并将确认异常数据输出到表。

## 记录方式

- 以业务数据单元为粒度生成 `msg_id`。
- 记录发送时间 `send_time` 与期望确认时间窗 `ack_deadline`。
- 捕获确认结果：
  - IEC104：I 帧确认（S 帧/接收序号）与 U 帧状态。
  - IEC101：链路层确认/否认与应用层确认（如有）。
- 记录最终状态 `ack_status`：`ack_ok`、`ack_nack`、`ack_timeout`、`ack_mismatch`。

## 异常判定

- 超时未确认：超过 `ack_deadline`。
- 否认确认：收到明确否认。
- 序号不匹配：确认序号与发送序号不一致。
- 重复确认：同一 `msg_id` 多次确认且结果冲突。

## 异常表输出

- 输出到 MySQL 表。
- 表名建议：`protocol_ack_exceptions`
- 关键字段：
  - `msg_id`
  - `protocol`（IEC101/IEC104）
  - `conn_id`
  - `send_time`
  - `ack_time`
  - `ack_status`
  - `cause`（timeout/nack/seq_mismatch/dup_ack）
  - `src_ip`、`dst_ip`、`src_port`、`dst_port`

## 异常状态关闭规则

- 收到与 `msg_id` 匹配的有效确认后关闭（`ack_status` 更新为 `ack_ok`）。
- 对于超时异常，超过最大保留窗口后关闭（如 24 小时），并标记为 `closed_timeout`。
- 若确认序号纠正（后续确认与发送序号一致），关闭并记录 `closed_recovered`。
- 关闭时写入 `close_time` 与 `close_reason`，便于追溯。
