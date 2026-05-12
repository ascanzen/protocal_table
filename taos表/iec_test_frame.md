# iec_test_frame（IEC协议测试帧表）

## 表结构设计

**表名**：`iec_test_frame`（TDengine超级表）

```sql
CREATE STABLE iec_test_frame (
    ts TIMESTAMP,                   -- 时间戳（精确到毫秒）
    frame_type VARCHAR(10),         -- 帧类型（U帧、S帧、I帧）
    control_function VARCHAR(20),   -- 控制功能（TESTFR、TESTFR_REPLY、STARTDT、STARTDT_REPLY、STOPDT、STOPDT_REPLY）
    control_byte1 TINYINT,          -- 控制域第一字节
    control_byte2 TINYINT,          -- 控制域第二字节
    control_byte3 TINYINT,          -- 控制域第三字节
    control_byte4 TINYINT,          -- 控制域第四字节
    original_data VARCHAR(100),     -- 原始数据（16进制字符串）
    source_ip VARCHAR(15),          -- 源IP
    source_port INT,                -- 源端口
    dest_ip VARCHAR(15),            -- 目标IP
    dest_port INT,                  -- 目标端口
    station_code VARCHAR(50),       -- 站点编码
    org_id VARCHAR(50),             -- 组织ID
    is_request BOOL,                -- 是否为请求帧（true=请求，false=响应）
    response_time INT               -- 响应时间（毫秒，仅响应帧有值）
) TAGS (
    device_address VARCHAR(50)      -- 设备地址（source_ip:source_port）
);
```

## 测试帧类型说明

| 控制功能 | 值 | 说明 | 用途 |
| ------- | --- | ---- | ---- |
| TESTFR | 0x43 | 测试命令 | 主站发送，测试链路是否正常 |
| TESTFR_REPLY | 0x83 | 测试确认 | 从站响应，确认链路正常 |
| STARTDT | 0x07 | 开启数据传输 | 主站发送，请求开始数据传输 |
| STARTDT_REPLY | 0x0B | 开启数据传输确认 | 从站响应，确认开始传输 |
| STOPDT | 0x13 | 关闭数据传输 | 主站发送，请求停止数据传输 |
| STOPDT_REPLY | 0x23 | 关闭数据传输确认 | 从站响应，确认停止传输 |

## 连接状态判断逻辑

IEC设备根据测试帧表来判断当前连接状态：

```text
设备在线判断规则：
1. 最近N分钟内有TESTFR或TESTFR_REPLY记录 → 设备在线
2. 最近N分钟内有STARTDT_REPLY记录 → 设备在线且数据传输正常
3. 最近N分钟内只有STOPDT_REPLY记录 → 设备在线但数据传输已停止
4. 超过N分钟无任何测试帧记录 → 设备离线或异常
（N值可配置，建议默认5分钟）
```
