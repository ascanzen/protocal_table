# channel_alarm_event（通道告警事件表）

## 表结构设计

**表名**：`channel_alarm_event`（TDengine超级表）

**设计思路**：基于通信通道维度的告警事件表，判据支持基于测试帧中断和PMU数据中断两种方式。与`device_alarm_event`的区别在于：device表关注单次协议事件，channel表关注通道级别的连续性中断判定。

```sql
CREATE STABLE channel_alarm_event (
    ts TIMESTAMP,                   -- 告警产生时间
    alarm_type VARCHAR(30),         -- 告警类型（TESTFR_INTERRUPT/PMU_DATA_INTERRUPT/PING_INTERRUPT）
    alarm_level VARCHAR(10),        -- 告警级别（WARN/ERROR/CRITICAL）
    alarm_desc VARCHAR(500),        -- 告警描述

    -- 判据信息
    judge_type VARCHAR(30),         -- 判据类型（TESTFR_BASED/PMU_BASED/PING_BASED）
    judge_threshold INT,            -- 判定阈值（秒）
    last_active_time TIMESTAMP,     -- 最后活跃时间（最后一次收到测试帧/PMU数据/Ping响应的时间）
    interrupt_duration INT,         -- 中断持续时长（秒）

    -- 业务信息
    device_name VARCHAR(100),       -- 设备名称
    station_code VARCHAR(50),       -- 站点编码
    org_id VARCHAR(50),             -- 组织ID

    -- 告警状态
    is_recovered BOOL,              -- 是否已恢复
    recover_time TIMESTAMP,         -- 恢复时间
    total_duration INT              -- 总中断时长（秒，恢复后计算）
) TAGS (
    source_ip VARCHAR(15),          -- 源IP
    source_port INT,                -- 源端口
    dest_ip VARCHAR(15),            -- 目标IP
    dest_port INT,                  -- 目标端口
    serial_port VARCHAR(50),        -- 串口号（IEC101使用）
    judge_category VARCHAR(20)      -- 判据分类（TESTFR/PMU/PING）
);
```

## 告警判据说明

### 判据一：基于测试帧中断

```text
判定逻辑：
跟踪TESTFR_ACT发送时间，15秒内无TESTFR_CON则触发CHANNEL_INTERRUPTION
或者：
  1. 定时查询iec_test_frame表中各设备的最新TESTFR_REPLY时间
  2. 若当前时间 - 最后TESTFR_REPLY时间 > 阈值（可配置，默认60秒）
  3. 则判定该通道测试帧中断，写入channel_alarm_event
  4. 当重新收到TESTFR_REPLY时，更新告警为已恢复

配置参数：
- testfr_timeout: 测试帧超时阈值（默认60秒）
- check_interval: 服务端检查周期（默认10秒）
```

### 判据二：基于PMU数据中断

```text
判定逻辑：
2秒滑动窗口计算帧率，低于25fps（期望50fps的50%）触发告警
或：
1. 定时查询pmu_data表中各设备的最新数据时间
2. 若当前时间 - 最后PMU数据时间 > 阈值（可配置，默认5秒）
3. 则判定该通道PMU数据中断，写入channel_alarm_event
4. 当重新收到PMU数据时，更新告警为已恢复

配置参数：
- pmu_data_timeout: PMU数据超时阈值（默认5秒）
- check_interval: 检查周期（默认2秒）
```

### 判据三：基于ICMP Ping中断

## 告警级别升级机制

```text
告警级别根据中断持续时长自动升级：
- WARN：中断时长 < 1分钟
- ERROR：中断时长 1~5分钟
- CRITICAL：中断时长 > 5分钟

级别升级时更新alarm_level字段，并可触发外部通知（如短信、邮件）。
告警抑制：同一通道同一判据类型，在告警未恢复期间不重复产生新告警。
```
