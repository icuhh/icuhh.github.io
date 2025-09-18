# 常见问题及解决方案记录

> 以下记录的问题是实际日常开发中遇到并处理过的BUG，或者经过优化的点。



## SQL集合参数过长导致堆栈内存溢出引发的错误

------

入参设置的集合元素太多了，6,7千个id循环匹配，可复现问题

    exception:org.springframework.web.util.NestedServletException: Handler dispatch failed; nested exception is java.lang.StackOverflowError

结果：切割分组查询优化后的代码：

    <if test="dto.orderNoList != null and dto.orderNoList.size() >0 ">
    <!--工单编号-->
    AND (t1.code in
      <foreach collection="dto.orderNoList" open="(" close=")"    separator="," item="orderNo" index="index">
            <if test="index !=0 and index % 999 == 0">
              #{orderNo}) or t1.code in(
            </if>
              #{orderNo}
       </foreach>
       )
     </if>

## 文件上传无文件报错问题

------

nacos配置问题，导致的服务降级失败引发的错误

```
exception:java.lang.IllegalArgumentException: Body parameter 0 was null
```

解决方法：经调整nacos ehr-service配置重启服务后正常

```
  hystrix:
    enabled: true
```

## 内存不足，启动过多项目引发的报错

------

```
Error:Abnormal build process termination:

D:\jdk1.8.0_202\bin\java.exe -Xmx2800m -Djava.awt.headless=true -Djava.endorsed.dirs=\"\" -Djdt.compiler.useSingleThread=true -Dexternal.project.config=C:\Users\Administrator\.IntelliJIdea2019.3\system\external_build_system\flinketl.3222d31f -Dcompile.parallel=false -Drebuild.on.dependency.change=true -Djava.net.preferIPv4Stack=true -Dio.netty.initialSeedUniquifier=-6871308861320460314 -Dfile.encoding=GBK -Duser.language=zh -Duser.country=CN -
```

解决方法：关掉其他项目，清理了内存后启动正常

## MySql 传输数据报错

------

当前 SQL 包的大小超过了服务器允许的最大值 *max_allowed_packet*。此问题通常发生在处理大数据插入或更新时。

```
[ERR] 1153 - Got a packet bigger than 'max_allowed_packet' bytes
[DTF] End transaction on target server
[DTF] Finished unsuccessfully
```

解决方法：

查看当前 *max_allowed_packet*的值，进入mysql运行以下命令查看当前设置,默认值可能为4MB（4194304）。

```
SHOW VARIABLES LIKE 'max_allowed_packet';
```

临时修改值

```
SET GLOBAL max_allowed_packet = 64 * 1024 * 1024; -- 设置为64MB
```

永久修改配置文件， 配置文件（如 *my.cnf* 或 *my.ini*），在 *[mysqld]* 部分添加或修改以下内容，保存后重启 MySQL 服务以生效

```
[mysqld]
max_allowed_packet=64M
```

如果客户端也需要处理大数据包，可以通过启动参数调整：

```
mysql --max_allowed_packet=64M
```

## 事务锁超时引发的报错

------

```
MySql Lock wait timeout exceeded
```

解决方法： 使用命令查看当前运行的所以事务

```
SELECT * FROM  information_schema.innodb_trx;
KILL 165667 ; 
```

KILL 后面的数字指的是 trx_mysql_thread_id 值。
