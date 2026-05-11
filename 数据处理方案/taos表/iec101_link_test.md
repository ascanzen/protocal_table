# iec101_link_test（IEC101链路测试帧表）

## 表结构设计

**表名**：`iec101_link_test`（TDengine超级表）

```sql
CREATE STABLE iec101_link_test (
    ts TIMESTAMP,                   -- 时间戳
    frame_type VARCHAR(10),         -- 帧类型（FIXED=固定帧，VARIABLE=可变帧）
    function_code TINYINT,          -- 功能码
    function_name VARCHAR(50),      -- 功能名称
    link_address INT,               -- 链路地址
    is_request BOOL,                -- 是否为请求帧
    response_time INT,              -- 响应时间（毫秒）
    serial_port VARCHAR(50),        -- 串口号（如COM1）
    station_code VARCHAR(50),       -- 站点编码
    org_id VARCHAR(50),             -- 组织ID
    original_data VARCHAR(200)      -- 原始数据
) TAGS (
    device_address VARCHAR(50)      -- 设备地址（串口+链路地址）
);
```

## IEC101功能码

| 功能码 | 名称 | 说明 |
| ------ | ---- | ---- |
| 0 | 复位远方链路 | 初始化链路 |
| 3 | 传送数据 | 发送用户数据 |
| 4 | 传送数据确认 | 确认接收数据 |
| 9 | 请求链路状态 | 链路测试 |
| 11 | 链路状态确认 | 链路测试响应 |
