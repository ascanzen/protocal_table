# station（站点与站点点表）

记录协议接入的站点信息及其下挂的四遥/PMU点号配置。

## 表结构设计

### t_station（站点表）

**表名**：`t_station`

存储站点基本信息，以 `station_code` 为业务唯一标识，关联组织（`org_id`）和业务类型。

```sql
DROP TABLE IF EXISTS `t_station`;
CREATE TABLE `t_station` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '站点ID',
  `station_code` varchar(255) NOT NULL DEFAULT '' COMMENT '站点编码',
  `org_id` varchar(255) DEFAULT NULL COMMENT '组织ID',
  `station_name` varchar(100) NOT NULL COMMENT '站点名称',
  `business_type` varchar(50) NOT NULL DEFAULT '' COMMENT '业务类型：REMOTE_CONTROL-远动，NEW_ENERGY-新能源，OTHER-其它',
  `description` varchar(255) DEFAULT NULL COMMENT '站点描述',
  `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '状态：0-停用，1-启用',
  `is_deleted` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否删除：0-未删除，1-已删除',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_station_code` (`station_code`),
  KEY `idx_org_id` (`org_id`),
  KEY `idx_business_type` (`business_type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='站点表';
```

### t_station_dot（站点点表）

**表名**：`t_station_dot`

存储站点下每个采集点的四遥类型、调度类型等配置。一条记录对应一个点号，通过 `station_id` 关联站点。

```sql
DROP TABLE IF EXISTS `t_station_dot`;
CREATE TABLE `t_station_dot` (
  `id` varchar(32) NOT NULL COMMENT '点ID',
  `station_id` bigint(20) DEFAULT NULL COMMENT '站点ID',
  `org_id` varchar(255) DEFAULT NULL COMMENT '组织ID',
  `dot_num` int(11) NOT NULL COMMENT '点号',
  `dot_name` varchar(255) DEFAULT NULL COMMENT '点名称',
  `type_code` varchar(20) NOT NULL COMMENT '四遥类型编码',
  `type_name` varchar(50) NOT NULL COMMENT '四遥类型名称',
  `business_type` varchar(50) NOT NULL DEFAULT '' COMMENT '业务类型：REMOTE_CONTROL-远动，NEW_ENERGY-新能源，OTHER-其它',
  `dispatch_type` varchar(50) NOT NULL DEFAULT 'LOCAL_DISPATCH' COMMENT '调度类型：CENTRAL_DISPATCH-中调，LOCAL_DISPATCH-地调',
  `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '状态：0-停用，1-启用',
  `is_deleted` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否删除：0-未删除，1-已删除',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_station_id` (`station_id`),
  KEY `idx_dot_num` (`dot_num`),
  KEY `idx_org_id` (`org_id`),
  KEY `idx_business_type` (`business_type`),
  KEY `idx_dispatch_type` (`dispatch_type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='站点点表';
```

## 表关系

```text
t_station (1) ── (N) t_station_dot

站点 : 点号 = 1 : N
一个站点下有多个采集点，每个点有独立的四遥类型和调度类型。
```

## 关键字段说明

| 字段 | 表 | 说明 |
| ---- | -- | ---- |
| `station_code` | t_station | 站点唯一编码，与 TDengine 表中的 `station_code` / `org_id` 对应 |
| `business_type` | 两表均有 | REMOTE_CONTROL（远动）、NEW_ENERGY（新能源）、OTHER（其它） |
| `dispatch_type` | t_station_dot | CENTRAL_DISPATCH（中调）、LOCAL_DISPATCH（地调） |
| `type_code` | t_station_dot | 四遥类型编码，与 four_remote_data 表中的遥测/遥信/遥控/遥调分类对应 |
| `dot_num` | t_station_dot | 点号，与 four_remote_data 的 TAG `dot_num` 对应 |
