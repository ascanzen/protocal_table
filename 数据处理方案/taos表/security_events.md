# security_events（网监设备安全事件表）

## 表结构设计

**表名**：`security_events`（TDengine超级表）

**设计思路**：记录IEC协议传输过程中的异常事件。

```sql
CREATE STABLE security_events (
    ts TIMESTAMP,                   -- 时间戳（精确到毫秒）
    common_address SMALLINT,        -- 公共地址
    severity INT,                   -- 级别（1-5，1最低，5最高）
    device VARCHAR(100),            -- 设备或系统/装置名称
    device_type VARCHAR(50),        -- 装置类型
    event_start_ts TIMESTAMP,       -- 事件开始时间戳（精确到毫秒）
    event_device_name VARCHAR(100), -- 事件设备名称（唯一标识）
    event_device_ip VARCHAR(15),    -- 事件设备IP（A网）
    event_type VARCHAR(50),         -- 事件类型（攻击、异常登录等）
    event_subtype VARCHAR(50),      -- 事件子类型（DDoS、端口扫描等）
    event_content VARCHAR(500),     -- 事件内容
    repeat_count INT,               -- 重复次数（同一事件触发次数）
    source_ip VARCHAR(15),          -- 源IP
    source_port INT,                -- 源端口
    dest_ip VARCHAR(15),            -- 目标IP
    dest_port INT,                   -- 目标端口
    station_code VARCHAR(50),        -- 站点编码
    org_id VARCHAR(50),             -- 组织ID（与station_code相同）
    protocol VARCHAR(10)             -- 协议类型（TCP、PMU、IEC104等）
) TAGS (
    event_device_type VARCHAR(50),  -- 事件设备类型（服务器/交换机/防火墙/监测装置）
    vendor VARCHAR(50)              -- 厂商名称（Cisco、华为等）
);
```

## 设备类型与事件类型

| 设备类型 | 行为监视事件 | 安全事件 |
| ------- | ---------- | ------- |
| 服务器(SVR) | 登录成功、退出登录、USB拔出、串口释放、并口释放、光驱卸载、设备上线 | USB插入、无线网卡插入、串口占用、并口占用、光驱挂载、外联事件、登录失败超阈值、关键文件变更、用户权限变更、危险操作、设备离线、开放非法端口、网口UP/DOWN |
| 交换机(SW) | MAC地址变更、上线、登录成功/失败、退出登录、修改密码、用户操作 | 网口UP/DOWN、流量超阈值、离线、端口未绑定MAC |
| 防火墙(FW) | 登录成功/失败、退出登录、修改策略、上线 | 不符合安全策略访问、攻击告警、离线、CPU超阈值 |
| 网络安全监测装置(DCD) | 系统登录成功、退出登录、USB拔出、管理界面登录/退出、配置变更 | USB插入、无线网卡插入、外联事件、登录失败超阈值、危险操作、开放非法端口、网口UP/DOWN、CPU/内存超阈值 |
