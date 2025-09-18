# 日常开发规约及需注意的点

> 关于一些项目开发中的规约





##  Service/DAO 层方法命名规约

------



1. 获取单个对象的方法用 `get` 做前缀
2. 获取多个对象的方法用 `list` 做前缀，复数结尾，如：`listobjects`
3. 获取统计值的方法用`count` 做前缀
4. 插入的方法用`save/insert`做前缀
5. 删除的方法用`remove/delete`做前缀
6. 修改的方法用`update`做前缀



## 领域模型命名规约

------



1. 数据对象：`xxxEntity`，xxx 即为数据表名，统一继承`BaseEntity`
2. 数据传输对象：`xxxDTO`，xxx 为业务领域相关的名称
3. 展示对象：`xxxVO`，xxx一般为网页名称
4. `POJO` 是`DO/DTO/BO/VO`的统称，禁止命名成`xxxPOJO`



## 数据库设定规约

------



1. 主键：类型为`bigint(20)`，对应实体类`Long`类型

2. 公共字段：

   ```
   　　　    `parent_id` bigint(20) NULL COMMENT '机构id',
   　　　    `create_by` bigint(20) NULL COMMENT '创建人',
   　　　    `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
   　　　    `update_by` bigint(20) NULL COMMENT '更新人',
   　　　    `update_time` datetime NULL COMMENT '更新时间',
   　　　    `is_delete` tinyint(1) DEFAULT '0' COMMENT '是否删除(0.否/1.是)'
   ```

3. 权限字段

   ​        个人发起：`create_by`

   ​	部分及子部门：`org_id`

   ​	本机构/相关机构：`parent_id`



## 其他规约

------



1. 控制层：加`@RefreshScope`支持nacos动态刷新
2. 分页方法返回：`R.ok(new PageInfo<>(listPage))`，listPage为分页查询返回集合



## 规范快照

------

![规范快照.png](/规范快照.png ':size=WIDTHxHEIGHT')





