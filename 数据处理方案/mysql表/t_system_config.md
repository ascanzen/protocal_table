# t_system_config（系统配置表）

存储系统级通用配置，支持按组织维度的 PMU 和基地址等参数配置。

## 表结构设计

**表名**：`t_system_config`

```sql
DROP TABLE IF EXISTS `t_system_config`;
CREATE TABLE `t_system_config` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `org_id` varchar(255) NOT NULL COMMENT '组织ID',
  `config_category` varchar(50) NOT NULL COMMENT '配置分类：pmu-PMU配置，base_address-基地址配置',
  `config_type` varchar(50) NOT NULL COMMENT '配置类型：phasor,analog,digital,yx,yc,yk,yt,ym',
  `config_value` varchar(500) NOT NULL COMMENT '配置值',
  `description` varchar(255) NOT NULL DEFAULT '' COMMENT '配置描述',
  `sort_order` int(11) DEFAULT '0' COMMENT '排序序号',
  `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '状态：0-停用，1-启用',
  `is_deleted` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否删除：0-未删除，1-已删除',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_org_category` (`org_id`, `config_category`),
  KEY `idx_config_type` (`config_type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='系统配置表';
```

## 配置分类说明

### config_category（配置分类）

| 分类 | 值 | 说明 |
| ---- | --- | ---- |
| PMU 配置 | `pmu` | PMU 测量装置的相关配置，如相量/模拟量/数字量的通道数、解析参数等 |
| 基地址配置 | `base_address` | IEC104/IEC101 协议的基地址配置，如公共地址范围、IOA 偏移等 |

### config_type（配置类型）

| 类型 | 适用分类 | 说明 |
| ---- | ------- | ---- |
| `phasor` | pmu | 相量通道配置（如通道数量、格式） |
| `analog` | pmu | 模拟量通道配置 |
| `digital` | pmu | 数字量通道配置 |
| `yx` | base_address | 遥信基地址配置 |
| `yc` | base_address | 遥测基地址配置 |
| `yk` | base_address | 遥控基地址配置 |
| `yt` | base_address | 遥调基地址配置 |
| `ym` | base_address | 遥脉基地址配置 |

## 配置使用方式

- 配置按 `org_id` + `config_category` + `config_type` 组合唯一确定一条配置。
- `sort_order` 用于同类型多条配置的排序（如多个相量通道的顺序）。
- 配置值 `config_value` 为字符串格式，具体含义由 `config_type` 决定。
- 停用的配置（`status=0`）不会被系统加载。
