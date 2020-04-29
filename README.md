<img src="https://user-images.githubusercontent.com/9434884/43697219-3cb4ef3a-9975-11e8-9a9c-73f4f537442d.png" alt="Sentinel Logo" width="50%">

# Sentinel: The Sentinel of Your Microservices

[![Travis Build Status](https://travis-ci.org/alibaba/Sentinel.svg?branch=master)](https://travis-ci.org/alibaba/Sentinel)
[![Codecov](https://codecov.io/gh/alibaba/Sentinel/branch/master/graph/badge.svg)](https://codecov.io/gh/alibaba/Sentinel)
[![Maven Central](https://img.shields.io/maven-central/v/com.alibaba.csp/sentinel-core.svg?label=Maven%20Central)](https://search.maven.org/search?q=g:com.alibaba.csp%20AND%20a:sentinel-core)
[![License](https://img.shields.io/badge/license-Apache%202-4EB1BA.svg)](https://www.apache.org/licenses/LICENSE-2.0.html)
[![Gitter](https://badges.gitter.im/alibaba/Sentinel.svg)](https://gitter.im/alibaba/Sentinel)

## 介绍

主要修改sentinel-dashboard模块来实现动态规则，通过控制台设置规则后将规则推送到统一的规则中心，客户端实现 ReadableDataSource 接口端监听规则中心实时获取变更。

DataSource 扩展常见的实现方式有2种：

- 拉模式：客户端主动向某个规则管理中心定期轮询拉取规则，这个规则中心可以是 RDBMS、文件，甚至是 VCS 等。这样做的方式是简单，缺点是无法及时获取变更；
- 推模式：规则中心统一推送，客户端通过注册监听器的方式时刻监听变化，比如使用 Nacos、Zookeeper 等配置中心。这种方式有更好的实时性和一致性保证。

这里使用推模式，同时数据源使用的是Nacos作为规则配置中心。


## 修改步骤

#### 1.修改父pom.xml文件

```xml
<dependency>
     <groupId>com.alibaba.csp</groupId>
     <artifactId>sentinel-datasource-nacos</artifactId>
<!-- <scope>test</scope> -->
</dependency>
```
#### 2.复制使用官方提供的相关类并修改

将```src/test/java/com/alibaba/csp/sentinel/dashboard/rule/nacos```文件复制到
```src/main/java/com/alibaba/csp/sentinel/dashboard/rule```下

具体修改可参考项目对应代码。

#### 3.修改 ```application.properties```

添加一下配置，配置可以自定义，主要用于NacosConfig中参数的使用
```java
# nacos地址
nacos.address=192.168.1.30:8848
# nacos的命名空间，默认使用public则无需配置
nacos.namespace=a0bc6fce-5b0a-47b3-8bfd-8c64eaf15862
```
#### 4.修改 ```src/main/java/com/alibaba/csp/sentinel/dashboard/controller/v2```下的controller

官方已经提供一个```FlowControllerV2.java```的限流规则的示例，根据示例添加降级熔断、热点等规则的controller即可。

具体修改可参考项目对应代码。

#### 5.修改控制台的页面

页面修改较为麻烦，以官方示例流控规则修改为例，主要关联以下文件修改，

- 1.修改 ```src/main/webapp/resources/app/scripts/directives/sidebar/sidebar.html```
    ```html
    <li ui-sref-active="active" ng-if="!entry.isGateway">
       <a ui-sref="dashboard.flowV1({app: entry.app})">
       <i class="glyphicon glyphicon-filter"></i>&nbsp;&nbsp;流控规则</a>
    </li>
    ```
    这里为了便于版本统一，修改```flowV1```为```flowV2```如下，官方原来是改为```flowV1```
    ```html
    <li ui-sref-active="active" ng-if="!entry.isGateway">
       <a ui-sref="dashboard.flowV2({app: entry.app})">
       <i class="glyphicon glyphicon-filter"></i>&nbsp;&nbsp;流控规则</a>
    </li>
    ```

- 2.修改```src/main/webapp/resources/app/scripts/app.js```，修改```flow```为```flowV2```，其他规则路由则无需改动。
  
- 3.添加```src/main/webapp/resources/app/views/flow_v2.html```，这里官方已经添加，无需改动，其他规则路由则无需改动。

- 4.添加```src/main/webapp/resources/app/scripts/services/flow_service_v2.js```，这里官方已经添加，无需改动，其他规则路由则无需改动。

- 5.添加```src/main/webapp/resources/app/scripts/controllers/flow_v2.js```，这里官方已经添加，无需改动，其他规则路由，
如果controller的地址改变则需要修改。

降级规则、热点规则等页面修改请参考项目对应代码。

## 结束

到此sentinel-dashboard的规则持久化完成，可以启动项目测试是否正常。

除此之外，动态规则修改生效还需要客户端添加nacos的配置中心监听代码来完成。