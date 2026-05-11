# four_remote_data（四遥数据表）

## 表结构设计

**表名**：`four_remote_data`（TDengine超级表）

```sql
CREATE STABLE four_remote_data (
    ts TIMESTAMP,                       -- 接收时间戳
    device_ts TIMESTAMP,                -- 设备时标（CP56Time2a，无时标类型为NULL）
    type_id TINYINT UNSIGNED,           -- 数据类型ID
    remote_type VARCHAR(10),            -- 四遥类型（YX/YC/YK/YT/YM）
    ioa INT,                            -- 信息对象地址

    -- 值字段
    value_float FLOAT,                  -- 浮点值（遥测浮点型/归一化转换值）
    value_int INT,                      -- 整数值（遥信SPI/DPI、遥测SVA/NVA原始值、遥脉BCR、遥控命令值）
    value_text VARCHAR(100),            -- 文本描述（"合"/"分"/"升档"/"降档"等）

    -- 品质（遥信SIQ/DIQ、遥测QDS、遥脉BCR品质）
    quality_descriptor TINYINT UNSIGNED, -- 品质原始字节
    is_valid BOOL,                      -- 有效性（IV=0有效，通用）

    -- 传输原因
    cause TINYINT UNSIGNED,             -- 传输原因（COT低6位）
    is_negative BOOL,                   -- P/N位：否定确认标志（bit6，true=否定）
    is_test BOOL,                       -- T位：测试标志（bit7）
    originator_address TINYINT UNSIGNED,-- 源发站地址（COT第2字节）

    -- 遥控/遥调特有
    se BOOL,                            -- 选择/执行标志（true=选择，false=执行）
    qu TINYINT UNSIGNED,                -- 命令限定词（遥控：0-3脉冲类型；遥调：QL）

    -- 网络与业务
    common_address SMALLINT,            -- 公共地址
    source_ip VARCHAR(45),
    source_port INT,
    dest_ip VARCHAR(45),
    dest_port INT,
    station_code VARCHAR(50),
    protocol_type VARCHAR(10)           -- IEC104/IEC101
) TAGS (
    org_id VARCHAR(50),
    device_address VARCHAR(80),
    dot_num INT,
    remote_category VARCHAR(10)
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
