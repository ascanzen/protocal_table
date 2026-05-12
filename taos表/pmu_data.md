# pmu_data（PMU数据表）

## 表结构设计

**表名**：`pmu_data`（TDengine超级表）

**设计思路**：一帧 PMU 数据对应一条记录，原始二进制数据以 `original_data` 存储。设备信息通过 TAGS 区分，支持按组织、设备、网络地址多维度查询。

```sql
CREATE STABLE pmu_data (
    ts TIMESTAMP,                   -- 时间戳（精确到毫秒）
    soc INT,                        -- 秒计数（SOC）
    fracsec INT,                    -- 微秒计数（FRACSEC）
    framesize INT,                  -- 帧大小
    idcode BIGINT,                  -- 设备标识码（可选，取决于协议版本）
    num_pmu INT,                    -- PMU测量装置个数
    original_data VARBINARY(800),   -- 原始数据
    chk INT                         -- CRC校验值
) TAGS (
    org_id VARCHAR(50),             -- 组织ID（与station_code相同）
    source_ip VARCHAR(15),          -- 源IP
    source_port INT,                -- 源端口
    dest_ip VARCHAR(15),            -- 目标IP
    dest_port INT,                  -- 目标端口
    device_id VARCHAR(36)           -- 设备ID
);
```

## 字段说明

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| `ts` | TIMESTAMP | 数据帧接收时间戳（毫秒精度） |
| `soc` | INT | 帧内秒计数，源自 PMU 数据帧时间标签 |
| `fracsec` | INT | 帧内微秒计数，与 soc 共同构成完整帧时间 |
| `framesize` | INT | 原始数据帧字节大小 |
| `idcode` | BIGINT | PMU 装置标识码，IEEE C37.118 协议定义，部分版本可能不携带 |
| `num_pmu` | INT | 该帧包含的 PMU 测量装置数量（一帧可含多个 PMU 数据） |
| `original_data` | VARBINARY(800) | 原始 PMU 数据帧字节，用于回放和二次解析 |
| `chk` | INT | CRC 校验和 |

### TAGS 说明

| TAG | 类型 | 说明 |
| --- | ---- | ---- |
| `org_id` | VARCHAR(50) | 组织标识，与 MySQL `t_station.org_id` 对应 |
| `source_ip` | VARCHAR(15) | PMU 数据源 IP 地址 |
| `source_port` | INT | PMU 数据源端口 |
| `dest_ip` | VARCHAR(15) | 目的 IP（采集服务器地址） |
| `dest_port` | INT | 目的端口（采集服务器监听端口） |
| `device_id` | VARCHAR(36) | 设备唯一 ID，用于关联设备信息 |

## 数据采集流程

```text
PMU装置 → UDP数据帧 → 网络抓包/采集服务 → 原始帧入库(pmu_data)
                                          → 解析 → 测量值JSON(measurements_data)
                                          → 告警判定 → channel_alarm_event（PMU中断判据）
```

## 注意事项

- 每子表对应一个唯一的 `source_ip:source_port` 组合，通过 TDengine 自动分表存储。
- `original_data` 最大 800 字节，超出部分会被截断（实际 PMU 帧一般不超过此大小）。
