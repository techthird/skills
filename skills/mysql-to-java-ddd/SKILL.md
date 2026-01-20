---
name: mysql-to-java-ddd
description:  MySQL表结构转Java Spring Boot DDD代码生成器
---
# Agent Skills - MySQL到Java Spring Boot DDD代码生成器

## 技能描述

根据MySQL表结构自动生成符合DDD(领域驱动设计)模式的Spring Boot项目代码,包含完整的API层、应用服务层、领域层和基础设施层,使用MyBatis Plus进行数据库操作。

## 功能范围

### 1. 支持的功能模块
- ✅ 新增(Create)
- ✅ 修改(Update)
- ✅ 根据ID查询(Get by ID)
- ✅ 分页查询(Page Query)

### 2. 生成的代码层次

#### API层 (entrance/api)
- SDK接口实现类
- 请求/响应DTO
- 参数校验

#### 应用服务层 (service)
- ApplicationService实现
- 业务编排逻辑
- 事务管理

#### 领域层 (domain)
- 聚合根(Aggregate)
- 值对象(ValueObject)
- 领域服务(DomainService)
- 仓储接口(Repository)
- 领域事件(Event)

#### 基础设施层 (infrastructure)
- 仓储实现(RepositoryImpl)
- MyBatis Plus Mapper
- DO(数据对象)转换

## 代码规范

### 1. 命名规范

#### 类命名
```
聚合根:      *Aggregate.java
值对象:      *ValueObject.java
领域服务:    *DomainService.java
应用服务:    *ApplicationService.java
仓储接口:    *Repository.java
仓储实现:    *RepositoryImpl.java
API接口:     *OperateApi.java / *QueryApi.java
DTO请求:     *Request.java
DTO响应:     *Response.java
DO对象:      *DO.java
Mapper:      *Mapper.java
```

#### 字段命名
- 遵循Java驼峰命名规范
- 数据库字段snake_case → Java字段camelCase
- 示例: `user_name` → `userName`

### 2. 注解规范

#### API层出参注解
```java
// 时间类型字段
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
private Date createTime;

// API文档注解
@ApiModelProperty(value = "支付中心支付单号")
private String payOrderNo;

@ApiModelProperty(value = "创建时间")
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
private Date createTime;
```

#### 入参校验注解
```java
@NotNull(message = "ID不能为空")
@ApiModelProperty(value = "主键ID", required = true)
private Long id;

@NotBlank(message = "订单号不能为空")
@ApiModelProperty(value = "订单号", required = true)
private String orderNo;
```

## 项目结构

### 整体架构
```
yxt-payment/
├── yxt-payment-server/        # 主应用模块 - 业务逻辑实现
├── yxt-payment-sdk/           # 内部SDK - 供内部系统调用
├── yxt-payment-open-sdk/      # 开放SDK - 供外部合作伙伴调用
├── yxt-payment-dao/           # 数据访问层
├── yxt-payment-common/        # 公共组件
└── yxt-payment-anticorruption/ # 防腐层 - 外部系统适配
```

### 核心包结构
```
com.yxt.payment.server.{domain}/
├── entrance/                  # 入口层
│   ├── api/                  # SDK接口实现(操作类)
│   │   └── *OperateApiImpl.java
│   ├── openapi/              # 开放API实现
│   ├── listener/             # MQ监听器
│   └── schedule/             # 定时任务
├── service/                   # 应用服务层
│   └── *ApplicationService.java
├── domain/                    # 领域层
│   ├── model/                # 聚合根和实体
│   │   ├── *Aggregate.java
│   │   └── *ValueObject.java
│   ├── service/              # 领域服务
│   │   └── *DomainService.java
│   ├── repository/           # 仓储接口
│   │   └── *Repository.java
│   ├── factory/              # 工厂类
│   │   └── *Factory.java
│   ├── event/                # 领域事件
│   └── listener/             # 事件监听器
└── infrastructure/            # 基础设施层
    ├── repository/           # 仓储实现
    │   └── *RepositoryImpl.java
    ├── gateway/              # 支付网关
    ├── producer/             # 消息生产者
    └── rest/                 # HTTP处理器
```

### SDK包结构
```
com.yxt.payment.sdk.{domain}/
├── *OperateApi.java          # 操作接口(新增、修改)
├── *QueryApi.java            # 查询接口(查询、分页)
├── request/                  # 请求DTO
│   ├── *CreateReqDTO.java
│   ├── *UpdateReqDTO.java
│   └── *PageQueryReqDTO.java
└── response/                 # 响应DTO
    ├── *ResDTO.java
    └── *PageResDTO.java
```

### DAO包结构
```
com.yxt.payment.dao.{domain}/
├── mapper/                   # MyBatis Mapper
│   └── *Mapper.java
└── dataobject/              # 数据对象
    └── *DO.java
```

## 技术栈

### 核心框架
- Spring Boot 2.x
- MyBatis Plus 3.x
- Lombok

### 必需依赖
```xml
<!-- MyBatis Plus -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
</dependency>

<!-- Lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>

<!-- Swagger注解 -->
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-annotations</artifactId>
</dependency>

<!-- Jackson -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
</dependency>
```

## 代码生成流程

### 输入要求
1. **MySQL DDL语句**
   ```sql
   CREATE TABLE `t_payment_order` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
     `order_no` varchar(64) NOT NULL COMMENT '订单号',
     `amount` decimal(10,2) NOT NULL COMMENT '金额',
     `status` tinyint(4) NOT NULL COMMENT '状态',
     `create_time` datetime NOT NULL COMMENT '创建时间',
     `update_time` datetime NOT NULL COMMENT '更新时间',
     PRIMARY KEY (`id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='支付订单表';
   ```

2. **业务域名称**
   - 示例: `pay`、`refund`、`reconcile`

3. **实体名称**
   - 示例: `PaymentOrder`、`RefundOrder`

### 输出内容

#### 1. SDK模块代码
```
yxt-payment-sdk/
└── src/main/java/com/yxt/payment/sdk/{domain}/
    ├── PaymentOrderOperateApi.java      # 操作接口
    ├── PaymentOrderQueryApi.java        # 查询接口
    ├── request/
    │   ├── PaymentOrderCreateReqDTO.java
    │   ├── PaymentOrderUpdateReqDTO.java
    │   └── PaymentOrderPageQueryReqDTO.java
    └── response/
        ├── PaymentOrderResDTO.java
        └── PaymentOrderPageResDTO.java
```

#### 2. Server模块代码
```
yxt-payment-server/
└── src/main/java/com/yxt/payment/server/{domain}/
    ├── entrance/api/
    │   ├── PaymentOrderOperateApiImpl.java
    │   └── PaymentOrderQueryApiImpl.java
    ├── service/
    │   └── PaymentOrderApplicationService.java
    ├── domain/
    |   ├── factory/
    │   │   └── PaymentOrderFactory.java
    │   ├── model/
    │   │   ├── PaymentOrderAggregate.java
    │   │   └── AmountValueObject.java
    │   ├── repository/
    │   │   └── PaymentOrderRepository.java
    │   └── service/
    │       └── PaymentOrderDomainService.java
    └── infrastructure/
        └── repository/
            └── PaymentOrderRepositoryImpl.java
```

#### 3. DAO模块代码
```
yxt-payment-dao/
└── src/main/java/com/yxt/payment/dao/{domain}/
    ├── mapper/
    │   └── PaymentOrderMapper.java
    └── dataobject/
        └── PaymentOrderDO.java
```

## DDD核心概念实现

### 1. 聚合根(Aggregate)
- 封装业务规则和不变性约束
- 作为事务一致性边界
- 包含业务方法(非贫血模型)

```java
public class PaymentOrderAggregate {
    private Long id;
    private String orderNo;
    private AmountValueObject amount;
    private Integer status;
    
    // 业务方法
    public void pay() {
        // 业务规则校验
        this.status = PaymentStatus.PAID;
    }
}
```

### 2. 值对象(ValueObject)
- 不可变对象
- 通过值比较相等性
- 无ID标识

```java
@Value
public class AmountValueObject {
    BigDecimal value;
    String currency;
}
```

### 3. 仓储(Repository)
- 领域层定义接口
- 基础设施层实现
- 面向聚合根操作

```java
// 领域层接口
public interface PaymentOrderRepository {
    void save(PaymentOrderAggregate aggregate);
    PaymentOrderAggregate findById(Long id);
}

// 基础设施层实现
public class PaymentOrderRepositoryImpl implements PaymentOrderRepository {
    @Autowired
    private PaymentOrderMapper mapper;
    // 使用MyBatis Plus实现
}
```

### 4. 应用服务(ApplicationService)
- 编排领域对象完成用例
- 管理事务
- 不包含业务逻辑

### 5. 领域服务(DomainService)
- 跨聚合根的业务逻辑
- 无状态
- 纯业务逻辑

## MyBatis Plus使用规范

### 1. Mapper接口
```java
public interface PaymentOrderMapper extends BaseMapper<PaymentOrderDO> {
    // 自定义复杂查询方法
    IPage<PaymentOrderDO> pageQuery(Page<PaymentOrderDO> page, 
                                     @Param("query") PaymentOrderPageQueryRequest query);
}
```

### 2. 基础CRUD
- 继承`BaseMapper`自动获得增删改查方法
- 使用`LambdaQueryWrapper`构建查询条件
- 使用`LambdaUpdateWrapper`构建更新条件

### 3. 分页查询
```java
Page<PaymentOrderDO> page = new Page<>(pageNum, pageSize);
LambdaQueryWrapper<PaymentOrderDO> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(PaymentOrderDO::getStatus, status);
Page<PaymentOrderDO> result = mapper.selectPage(page, wrapper);
```

## 接口实现规范

### 操作类接口(OperateApi)
```java
public interface PaymentOrderOperateApi {
    /**
     * 创建支付订单
     */
    Result<Long> create(PaymentOrderCreateRequest request);
    
    /**
     * 更新支付订单
     */
    Result<Boolean> update(PaymentOrderUpdateRequest request);
}
```

### 查询类接口(QueryApi)
```java
public interface PaymentOrderQueryApi {
    /**
     * 根据ID查询
     */
    Result<PaymentOrderResponse> getById(Long id);
    
    /**
     * 分页查询
     */
    Result<PageResult<PaymentOrderResponse>> pageQuery(PaymentOrderPageQueryRequest request);
}
```

## 数据转换规范

### 转换链路
```
Request DTO → Aggregate → DO → Database
Database → DO → Aggregate → Response DTO
```

### 转换器实现
```java
// 使用工厂或转换器类
public class PaymentOrderFactory {
    public static PaymentOrderAggregate fromDO(PaymentOrderDO dataObject) {
        // DO转聚合根
    }
    
    public static PaymentOrderDO toDO(PaymentOrderAggregate aggregate) {
        // 聚合根转DO
    }
    
    public static PaymentOrderResponse toResponse(PaymentOrderAggregate aggregate) {
        // 聚合根转响应DTO
    }
}
```

## 事务管理

### 应用服务层添加事务
```java
@Service
public class PaymentOrderApplicationService {
    
    @Transactional(rollbackFor = Exception.class)
    public Long createOrder(PaymentOrderCreateRequest request) {
        // 业务逻辑
    }
}
```

## 异常处理

### 异常层次
```
BizException (业务异常)
├── ValidationException (参数校验异常)
├── NotFoundException (资源不存在异常)
└── StatusException (状态异常)
```

## 代码质量要求

### 1. 代码风格
- 遵循阿里巴巴Java开发手册
- 所有类和方法必须有清晰的Javadoc注释
- 方法长度不超过50行

### 2. 必需注解
- `@Slf4j` - 日志
- `@Data` / `@Value` - Lombok
- `@Service` / `@Repository` - Spring组件
- `@Transactional` - 事务管理
- `@ApiModelProperty` - API文档

### 3. 最佳实践
- 使用`Optional`处理可能为空的返回值
- 使用`@NonNull`标注非空参数
- 使用枚举代替魔法值
- 合理使用设计模式(工厂、策略等)

## 使用示例

### 输入
```
业务域: pay
实体名: PaymentOrder
表名: t_payment_order

CREATE TABLE `t_payment_order` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `order_no` varchar(64) NOT NULL,
  `amount` decimal(10,2) NOT NULL,
  `status` tinyint(4) NOT NULL,
  `create_time` datetime NOT NULL,
  `update_time` datetime NOT NULL,
  PRIMARY KEY (`id`)
);
```

### 输出
生成完整的DDD分层代码,包括:
- 2个SDK接口(OperateApi、QueryApi)
- 6个DTO类(3个Request、3个Response)
- 2个API实现类
- 1个ApplicationService
- 1个Aggregate
- 1个Repository接口
- 1个RepositoryImpl实现
- 1个Mapper接口
- 1个DO类

## 注意事项

1. ⚠️ 表名必须符合规范(建议`t_`前缀)
2. ⚠️ 主键必须为`id`,类型为`bigint`
3. ⚠️ 必须包含`create_time`和`update_time`字段
4. ⚠️ 字段COMMENT必须填写,用于生成API注解
5. ⚠️ 生成代码后需要手动添加业务逻辑到聚合根
6. ⚠️ 复杂查询需要在Mapper XML中编写SQL
7. ⚠️ 需要根据实际业务调整领域服务和应用服务

## 扩展能力

### 可选生成内容
- 领域事件及监听器
- MQ消息生产者
- 定时任务
- 开放API实现
- 防腐层适配器
- 单元测试用例

---

**版本**: v1.0  
**最后更新**: 2025-01-20  
**适用框架**: Spring Boot 2.x + MyBatis Plus 3.x