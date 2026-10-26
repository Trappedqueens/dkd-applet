# 售货机小程序后端 · dkd-applet

售货机小程序对应的 Spring Boot 后端学习工程。目前主要实现按设备编号查询商品，并结合设备关联的策略计算商品实际价格。

这是 **Java 后端服务**，不包含微信小程序页面源码。支付接口目前仍是占位实现。

## 当前功能

| 功能 | 状态 |
| --- | --- |
| 按售货机编号查询商品 | 已有控制器和服务实现 |
| 查询售货机及关联策略 | 已有服务实现 |
| 根据折扣率计算实际价格 | 在 `SkuServiceImpl` 中实现 |
| 微信/支付宝支付依赖 | POM 中已声明相关 SDK |
| 发起支付接口 | `OrderController.requestPay` 当前直接返回 `null`，尚未完成 |

依赖中出现支付 SDK 不等于支付链路可用，也不代表已实现真实订单、支付回调或出货闭环。

## 技术栈

- Java 11、Spring Boot 2.7.3。
- MyBatis / MyBatis-Plus、MySQL、Druid、PageHelper。
- Redis、Hutool、Lombok、Knife4j。
- 项目声明的第三方支付 SDK。

## 目录

```text
src/main/java/com/dkd/applet/
├─ Application.java
├─ controller/       设备商品与支付入口
├─ service/impl/     商品、策略、售货机服务
├─ mapper/           数据访问
└─ domain/           实体、DTO 与 VO
src/main/resources/
├─ application.yml
└─ application-dev.yml
```

## 环境准备

1. 准备 **JDK 11**、Maven、MySQL 和 Redis。
2. 通过 [application.yml](src/main/resources/application.yml) 和 [application-dev.yml](src/main/resources/application-dev.yml) 配置实际数据库与 Redis。
3. 默认启用 `dev`，数据库连接通过 `dkd.datasource` 参数引用，Redis 连接参数见对应环境配置。
4. 准备与 Mapper 匹配的售货机、商品、货道、策略等数据库结构和关联数据。本仓库没有 SQL 初始化脚本。
5. 查询使用的设备编号必须在数据库中存在，并具有相应的商品/货道及策略数据。

不要在仓库里提交真实数据库密码、支付商户信息、私钥或证书。

## 启动

```shell
git clone https://github.com/Trappedqueens/dkd-applet.git
cd dkd-applet
mvn spring-boot:run
```

这是私有仓库，克隆需要有权限的 GitHub 账号。

默认端口为 **9008**。构建运行：

```shell
mvn clean package -DskipTests
java -jar target/dkd-applet.jar
```

也可以在 IDE 中运行 `com.dkd.applet.Application`。

## 接口

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| 按控制器映射 | `/vm/skuList/{innerCode}` | 根据售货机编号查询商品及价格 |
| POST | `/order/requestPay/{tradeType}/{platform}` | 支付占位入口，当前返回 `null` |

商品查询入口使用 `@RequestMapping`，当前未限定单独的 HTTP 方法，可按 GET 请求进行对接。完整数据字段见 [controller](src/main/java/com/dkd/applet/controller) 和 [domain](src/main/java/com/dkd/applet/domain)。

## 价格与支付说明

- 价格计算使用售货机关联的策略折扣率；无关联策略时，代码使用默认折扣率。
- 字段单位和取整行为以实体、数据库及当前代码为准，对接页面前需要核对。
- 支付接口尚未完成，不应当作可运行支付示例，也不应直接用于真实收款。
- 需要实际支付能力时，应另行补齐订单持久化、支付参数、通知验签、状态处理及必要的幂等校验。

本次只添加项目说明，没有补实现支付功能，也没有运行构建、访问真实数据库或发起支付。
