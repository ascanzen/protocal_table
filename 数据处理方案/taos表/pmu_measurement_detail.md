# pmu_measurement_detail（PMU数据表）

## 表结构设计（扁平化存储）

**表名**：`pmu_measurement_detail`（TDengine超级表）

**设计思路**：将原 `measurements_data` JSON字段拆分为独立列，便于查询、统计和分析。

```sql
CREATE STABLE pmu_measurement_detail (
    ts TIMESTAMP,                   -- 时间戳（精确到毫秒）
    measurement_idx INT,          -- 测量装置索引（0, 1, 2...，一帧可能包含多个PMU）

    -- 状态与频率数据
    stat INT,                       -- 状态字（16位完整值）
    freq SMALLINT,                  -- 频率偏移（原始值，除以1000得Hz）
    dfreq SMALLINT,                 -- 频率变化率（原始值，除以100得Hz/s）

    -- 状态字解析字段（便于直接查询）
    data_valid BOOL,                -- 数据有效性（Bit 15，0=有效，1=无效）
    pmu_normal BOOL,                -- PMU设备状态（Bit 14，0=正常，1=异常）
    pmu_sync BOOL,                  -- PMU同步状态（Bit 13，0=同步，1=失步）
    -- 动态部分用 JSON 存
    phasors         VARCHAR(4000),   -- JSON: [{mag:..., angle:...}, ...]
    analogs         VARCHAR(2000),   -- JSON: [val1, val2, ...]
    digitals        VARCHAR(500),    -- JSON: [word1, word2, ...]

    -- 网络信息
    source_ip VARCHAR(15),          -- 源IP
    source_port INT,                -- 源端口
    dest_ip VARCHAR(15),            -- 目标IP
    dest_port INT,                  -- 目标端口
) TAGS (
    idcode          BIGINT,
    source_ip_port  VARCHAR(50),
    station_code    VARCHAR(50),
    org_id          VARCHAR(50)
);
```
