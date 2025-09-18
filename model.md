# 日常开发一些复杂需求的分析及拆分调整记录

> 关于一些复杂的调整的都记录在此



## 维修多方案相关的调整

------

1. EMC 检修流程相关的调整：

​              一个检修流程可以有多个检测记录，一个检测记录可以添加多个维修建议。原本检修记录表里的部分字段抽离了出来，创建了

维修多方案表`t_emc_maintain_plan`，并与原本的检修记录表`t_emc_inspection_record`进行挂钩，一对多模式。

|            id             | bigint   | 主键id                                                       |
| :-----------------------: | -------- | ------------------------------------------------------------ |
|   inspection_record_id    | bigint   | 检修记录表id                                                 |
|       equipment_id        | bigint   | 设备id                                                       |
|      inspection_date      | datetime | 检测完成时间                                                 |
|    inspection_user_id     | bigint   | 检测人id                                                     |
|   inspection_user_name    | varchar  | 检测人名称                                                   |
|           fault           | int      | 是否有故障（1.正常无故障，2有故障）检修类型                  |
|    third_party_repair     | int      | 是否有其他三方维修（1.被其他方修过，2.未被其他方修过）       |
|      hidden_trouble       | int      | 是否有隐患（1.无隐患，2.需维修后确认是否有隐患，3.已发现隐患 |
|      fault_category       | varchar  | 故障分类                                                     |
| maintenance_result_detail | text     | 检测结果详情                                                 |
|          remark           | varchar  | 简化显示文字                                                 |
|     inspection_record     | text     | 检测记录                                                     |
|     repair_suggestion     | text     | 维修建议                                                     |
|   estimate_repair_time    | varchar  | 预计维修时长/D                                               |
|        repair_plan        | text     | 维修方案                                                     |
|        is_checked         | tinyint  | 当前方案是否被选中（0.否，1.是）                             |
|     purchase_content      | varchar  | 拆解的采购内容                                               |
|        supplier_id        | bigint   | 供应商id                                                     |
|       supplier_name       | varchar  | 供应商名称                                                   |
|    supplier_contact_id    | bigint   | 供应商联系人id                                               |
|   supplier_contact_name   | varchar  | 供应商联系人名称                                             |
|       plan_group_id       | bigint   | 分组序号（维修多方案分组用,同组记录序号一致）                |
|       is_take_apart       | int      | 是否采购模块拆解维修方案生成（1.未拆解，2.已拆解，3.拆解后的数据） |
|         data_type         | tinyint  | 数据类型（0.维修方案，1.拆解的采购内容）                     |

通过 `inspection_record_id`字段与检修记录表关联，通过`id`与设备检修表`maintain_plan_id`字段和`plan_group_id`字段关联。



2. ERP技术服务采购单相关的调整：

​		采购`t_purchase_inquiry`询价记录表通过 维修建议id`maintain_plan_id`字段，与维修多方案建议表关联，一条维修建议对应采购一条询价记录。



3. 基础合同相关调整：

​		





