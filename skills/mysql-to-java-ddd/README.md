# MySQL表结构转Java Spring Boot DDD代码生成器

# 使用方式 
```
/mysql-to-java-ddd
  
业务域: merchant
实体名: ApplicationManage
表名: application_manage

CREATE TABLE `application_manage` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `merchant_code` varchar(50) NOT NULL COMMENT '商户编码（唯一标识）',
  `merchant_name` varchar(100) NOT NULL COMMENT '商户名称'
  -- 省略N个字段
  PRIMARY KEY (`id`)
);
```