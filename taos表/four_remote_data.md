# four_remote_data（四遥数据表）

## 表结构设计

**表名**：`four_remote_data`（TDengine超级表）

```sql
CREATE STABLE four_remote_data (
    ts TIMESTAMP,     -- 时间戳（精确到毫秒）
    info_obj_address VARCHAR(50),   -- 信息对象地址
    info_obj_value VARCHAR(5000)     -- 信息对象值
) TAGS (
    org_id VARCHAR(50),             -- 组织ID（与station_code相同）
    source_ip VARCHAR(15),          -- 源IP
    source_port INT,                -- 源端口
    dest_ip VARCHAR(15),            -- 目标IP
    dest_port INT,                  -- 目标端口
    common_address SMALLINT,        -- 公共地址
    type_id INT,                    -- 数据类型ID
    dot_num INT,                    -- 点号
    teleType VARCHAR(20),           -- 四遥类型（遥测、遥信、遥控、遥调）
    device_id VARCHAR(36),          -- 设备ID
    business_type VARCHAR(50),      -- 业务类型
    dispatch_type VARCHAR(50)       -- 调度类型
);
```

## 四遥类型映射（基于IEC104标准）

| 四遥类型 | 代码 | IEC104类型ID | 类型名称 | 说明 |
| ------- | ---- | ----------- | ------- | ---- |
| **遥测（YC）** | YC | 9 | M_ME_NA_1 | 归一化测量值 |
| | | 11 | M_ME_NB_1 | 标量化测量值 |
| | | 13 | M_ME_NC_1 | 浮点型测量值 |
| | | 21 | M_ME_ND_1 | 不带品质描述的归一化测量值 |
| | | 34 | M_ME_TD_1 | 带时标CP56Time2a的归一化测量值 |
| | | 35 | M_ME_TE_1 | 带时标CP56Time2a的标量化测量值 |
| | | 36 | M_ME_TF_1 | 带时标CP56Time2a的浮点型测量值 |
| **遥信（YX）** | YX | 1 | M_SP_NA_1 | 单点信息 |
| | | 3 | M_DP_NA_1 | 双点信息 |
| | | 30 | M_SP_TB_1 | 带时标CP56Time2a的单点信息 |
| | | 31 | M_DP_TB_1 | 带时标CP56Time2a的双点信息 |
| | | 20 | M_PS_NA_1 | 带状态检出的成组单点信息 |
| **遥控（YK）** | YK | 45 | C_SC_NA_1 | 单命令 |
| | | 46 | C_DC_NA_1 | 双命令 |
| | | 47 | C_RC_NA_1 | 步调节命令 |
| | | 58 | C_SC_TA_1 | 带时标CP56Time2a的单命令 |
| | | 59 | C_DC_TA_1 | 带时标CP56Time2a的双命令 |
| | | 60 | C_RC_TA_1 | 带时标CP56Time2a的步调节命令 |
| **遥调（YT）** | YT | 48 | C_SE_NA_1 | 设点命令, 归一化值 |
| | | 49 | C_SE_NB_1 | 设点命令, 标量值 |
| | | 50 | C_SE_NC_1 | 设点命令, 短浮点值 |
| | | 61 | C_SE_TA_1 | 带时标CP56Time2a的设点命令, 归一化值 |
| | | 62 | C_SE_TB_1 | 带时标CP56Time2a的设点命令, 标量值 |
| | | 63 | C_SE_TC_1 | 带时标CP56Time2a的设点命令, 短浮点值 |
